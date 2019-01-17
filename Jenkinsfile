@Library('dynatrace@master') _

def tagMatchRules = [
  [
    meTypes: [
      [meType: 'SERVICE']
    ],
    tags : [
      [context: 'CONTEXTLESS', key: 'app', value: ''],
      [context: 'CONTEXTLESS', key: 'environment', value: 'staging']
    ]
  ]
]

pipeline {
  parameters {
    string(name: 'APP_NAME', defaultValue: '', description: 'The name of the service to deploy.', trim: true)
    //string(name: 'TAG_STAGING', defaultValue: '', description: 'The image of the service to deploy.', trim: true)
    //string(name: 'TAG_DEV', defaultValue: '', description: '.', trim: true)
    //string(name: 'VERSION', defaultValue: '', description: 'The version of the service to deploy.', trim: true)
  }
  agent {
    label 'kubegit'
  }
  stages {
    stage('Check out config') {
      steps {
        container('git') {
          withCredentials([usernamePassword(credentialsId: 'git-credentials-acm', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
            sh "git config --global user.email ${env.GITHUB_USER_EMAIL}"
            sh "git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/config"
            sh "cd config && git checkout staging"
          }
        }
      }
    }
    stage('Deploy to staging namespace') {
      steps {
        //checkout scm
        container('kubectl') {
          sh "cd config && kubectl -n staging apply -f ."
        }
      }
    }  
    stage('DT Deploy Event') {
      steps {
        container("curl") {
          script {
            tagMatchRules[0].tags[0].value = "${env.APP_NAME}"
            def status = pushDynatraceDeploymentEvent (
              tagRule : tagMatchRules,
              customProperties : [
                [key: 'Jenkins Build Number', value: "${env.BUILD_ID}"],
                //[key: 'Git commit', value: "${env.GIT_COMMIT}"]
              ]
            )
          }
        }
      }
    }
    /*
    stage('Run production ready e2e check in staging') {
      steps {
        echo "Waiting for the service to start..."
        sleep 150

        recordDynatraceSession (
          envId: 'Dynatrace Tenant',
          testCase: 'loadtest',
          tagMatchRules: tagMatchRules
        ) 
        {
          container('jmeter') {
            script {
              def status = executeJMeter ( 
                scriptName: "jmeter/front-end_e2e_load.jmx",
                resultsDir: "e2eCheck_${env.APP_NAME}",
                serverUrl: "front-end.staging", 
                serverPort: 8080,
                checkPath: '/health',
                vuCount: 10,
                loopCount: 5,
                LTN: "e2eCheck_${BUILD_NUMBER}",
                funcValidation: false,
                avgRtValidation: 4000
              )
              if (status != 0) {
                currentBuild.result = 'FAILED'
                error "Production ready e2e check in staging failed."
              }
            }
          }
        }

        perfSigDynatraceReports(
          envId: 'Dynatrace Tenant', 
          nonFunctionalFailure: 1, 
          specFile: "monspec/e2e_perfsig.json"
        )
      }
    }
    */
    stage('Promote deployment and artifact for stable') {
      steps {
        container('docker'){
          sh "docker pull ${env.TAG_STAGING}"
          sh "docker tag ${env.TAG_STAGING} ${env.APP_NAME}:staging-stable"
          sh "docker push ${env.TAG_STAGING}"
        }
      }
    }
    stage('Get Artifact ID and promote deployment for stable') { // and mark artifact as staging-passed
      steps {
        container('docker'){
          sh "cd config && cat ${env.APP_NAME}.yml | grep image: | sed 's/[ \t]*image:[ \t]*//' > image.txt"
          script {
            IMAGE_TAG = readFile('config/image.txt').trim()
            //PASSED_TAG = 
            PULL_REQUEST = IMAGE_TAG
          }
          sh "echo ${IMAGE_TAG}"
          sh "docker pull ${IMAGE_TAG}"
          sh "docker tag ${IMAGE_TAG} ${env.APP_NAME}:staging-stable"
          sh "docker push ${IMAGE_TAG}"
        }
      }
    }
  }
  post {
    always {
      container("curl") {
        sh "curl -X POST \"https://us-central1-sai-research.cloudfunctions.net/jenkinsNotificationListener\" -H \"accept: application/json\" -H \"Content-Type: application/json\" -d \"{ \\\"status\\\": \\\"success\\\", \\\"environment\\\": \\\"staging\\\", \\\"service\\\": \\\"${env.APP_NAME}\\\", \\\"pull_request\\\": \\\"${PULL_REQUEST}\\\"}\" "       
      }
    }
  }
}