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

def IMAGE_TAG = 'UNKNOWN'
def PULL_REQUEST = 'UNKNOWN'
def STABLE_TAG = 'UNKNOWN'

pipeline {
  parameters {
    string(name: 'APP_NAME', defaultValue: '', description: 'The name of the service to deploy.', trim: true)
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
            sh "git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/keptn-config"
            sh "cd keptn-config && git checkout staging"
          }
        }
      }
    }
    stage('Deploy to staging namespace and apply istio config') {
      steps {
        container('helm') {
          sh "helm init --client-only"
          sh "cd keptn-config && helm dep update helm-chart/"
          sh "cd keptn-config && helm upgrade --install ${env.GITHUB_ORGANIZATION}-staging ./helm-chart --namespace staging"
        }
        /*
        container('kubectl') {
          sh "cd keptn-/istio && kubectl apply -f ."
        }
        */
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
    stage('Get artifact ID and promote deployment as staging-passed') {
      steps {
        container('yq') {
          sh "cd keptn-config/helm-chart && yq r values.yaml ${env.APP_NAME}Green.image.tag > image-tag.txt"
          sh "cd keptn-config/helm-chart && yq r values.yaml ${env.APP_NAME}Green.image.repository > image-repository.txt"
          script {
            IMAGE_REPOSITORY = readFile('keptn-config/helm-chart/image-repository.txt').trim().toLowerCase()
            IMAGE_TAG = readFile('keptn-config/helm-chart/image-tag.txt').trim()
            PULL_REQUEST = IMAGE_REPOSITORY + ':' + IMAGE_TAG
            STABLE_TAG = IMAGE_REPOSITORY + ':staging-stable'
          }   
        }
        container('docker'){
          sh "echo ${IMAGE_TAG}"
          sh "echo ${STABLE_TAG}"
          sh "docker pull ${IMAGE_REPOSITORY}:${IMAGE_TAG}"
          sh "docker tag ${IMAGE_REPOSITORY}:${IMAGE_TAG} ${STABLE_TAG}"
          sh "docker push ${STABLE_TAG}"
        }
      }
    }
  }
  post {
    always {
      container("curl") {
        sh "curl -X POST \"https://us-central1-sai-research.cloudfunctions.net/jenkinsNotificationListener\" -H \"accept: application/json\" -H \"Content-Type: application/json\" -d \"{ \\\"status\\\": \\\"success\\\", \\\"environment\\\": \\\"staging\\\", \\\"service\\\": \\\"${env.APP_NAME}\\\", \\\"github_org\\\": \\\"${env.GITHUB_ORGANIZATION}\\\", \\\"pull_request\\\": \\\"${PULL_REQUEST}\\\"}\" "       
      }
    }
  }
}
