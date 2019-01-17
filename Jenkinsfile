@Library('dynatrace@master') _

def tagMatchRules = [
  [
    meTypes: [
      [meType: 'SERVICE']
    ],
    tags : [
      [context: 'CONTEXTLESS', key: 'app', value: ''],
      [context: 'CONTEXTLESS', key: 'environment', value: 'production']
    ]
  ]
]

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
            sh "git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/config"
            sh "cd config && git checkout production"
          }
        }
      }
    }
    stage('Deploy to production namespace and apply istio config') {
      steps {
        //checkout scm
        container('kubectl') {
          sh "cd config && kubectl -n production apply -f ."
          sh "cd config/istio && kubectl apply -f ."
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
    stage('Get artifact ID and promote deployment for stable') { // and mark artifact as production-passed
      steps {
        container('docker'){
          sh "cd config && cat ${env.APP_NAME}.yml | grep image:.*#image-green | sed 's/[ \t]*image:[ \t]*//' | sed 's/[ \t]*#image-green//' > image.txt"
          script {
            IMAGE_TAG = readFile('config/image.txt').trim() //10.43.249.155:5000/library/sockshop/carts:pr-31
            PULL_REQUEST = IMAGE_TAG
            //println(IMAGE_TAG)
            
            def array = IMAGE_TAG.split(':')
            STABLE_TAG = ''

            for (i = 0; i < array.length-1; i++) {
              STABLE_TAG += array[i]
              STABLE_TAG += ':'
            }
            STABLE_TAG += 'production-stable'

            //println(STABLE_TAG)
          }
          sh "echo ${IMAGE_TAG}"
          sh "echo ${STABLE_TAG}"
          sh "docker pull ${IMAGE_TAG}"
          sh "docker tag ${IMAGE_TAG} ${STABLE_TAG}"
          sh "docker push ${STABLE_TAG}"
        }
      }
    }
  }
  post {
    always {
      container("curl") {
        sh "curl -X POST \"https://us-central1-sai-research.cloudfunctions.net/jenkinsNotificationListener\" -H \"accept: application/json\" -H \"Content-Type: application/json\" -d \"{ \\\"status\\\": \\\"success\\\", \\\"environment\\\": \\\"production\\\", \\\"service\\\": \\\"${env.APP_NAME}\\\", \\\"pull_request\\\": \\\"${PULL_REQUEST}\\\"}\" "       
      }
    }
  }
}