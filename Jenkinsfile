@Library('dynatrace@master') _

def tagMatchRules = [
  [
    meTypes: [
      [meType: 'SERVICE']
    ],
    tags : [
      [context: 'CONTEXTLESS', key: 'app', value: ''],
      [context: 'CONTEXTLESS', key: 'environment', value: 'dev']
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
            sh "cd keptn-config && git checkout dev"
          }
        }
      }
    }
    stage('Deploy to dev namespace') {
      steps {
        container('helm') {
          sh "cd keptn-config && helm upgrade --install ${env.GITHUB_ORGANIZATION} ./helm-chart"
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
    stage('Run health check in dev') {
      steps {
        echo "Waiting for the service to start..."
        sleep 150

        container('jmeter') {
          script {
            def status = executeJMeter ( 
              scriptName: 'jmeter/basiccheck.jmx', 
              resultsDir: "HealthCheck_${env.APP_NAME}",
              serverUrl: "${env.APP_NAME}.dev", 
              serverPort: 80,
              checkPath: '/health',
              vuCount: 1,
              loopCount: 1,
              LTN: "HealthCheck_${BUILD_NUMBER}",
              funcValidation: true,
              avgRtValidation: 0
            )
            if (status != 0) {
              currentBuild.result = 'FAILED'
              error "Health check in dev failed."
            }
          }
        }
      }
    }
    stage('Run functional check in dev') {
      steps {
        container('jmeter') {
          script {
            def status = executeJMeter (
              scriptName: "jmeter/${env.APP_NAME}_load.jmx", 
              resultsDir: "FuncCheck_${env.APP_NAME}",
              serverUrl: "${env.APP_NAME}.dev", 
              serverPort: 80,
              checkPath: '/health',
              vuCount: 1,
              loopCount: 1,
              LTN: "FuncCheck_${BUILD_NUMBER}",
              funcValidation: true,
              avgRtValidation: 0
            )
            if (status != 0) {
              currentBuild.result = 'FAILED'
              error "Functional check in dev failed."
            }
          }
        }
      }
    }
    */
    stage('Get artifact ID and mark deployment as dev-stable') { 
      steps {
        container('yq') {
          sh "cd keptn-config/helm-chart && yq r values.yml ${env.APP_NAME}.image.tag' > image-tag.txt"
          sh "cd keptn-config/helm-chart && yq r values.yml ${env.APP_NAME}.image.repository' > image-repository.txt"
          script {
            IMAGE_REPOSITORY = readFile('keptn-config/helm-chart/image-repository.txt').trim()
            IMAGE_TAG = readFile('keptn-config/helm-chart/image-tag.txt').trim()
            PULL_REQUEST = IMAGE_REPOSITORY + ':' + IMAGE_TAG
            STABLE_TAG = IMAGE_REPOSITORY + ':dev-stable'
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
        sh "curl -X POST \"https://us-central1-sai-research.cloudfunctions.net/jenkinsNotificationListener\" -H \"accept: application/json\" -H \"Content-Type: application/json\" -d \"{ \\\"status\\\": \\\"success\\\", \\\"environment\\\": \\\"dev\\\", \\\"service\\\": \\\"${env.APP_NAME}\\\", \\\"github_org\\\": \\\"${env.GITHUB_ORGANIZATION}\\\", \\\"pull_request\\\": \\\"${PULL_REQUEST}\\\"}\" "       
      }
    }
  }
}
