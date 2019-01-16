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
            sh "cd config && git checkout dev"
          }
        }
      }
    }
    stage('Deploy to dev namespace') {
      steps {
        //checkout scm
        container('kubectl') {
          sh "cd config && kubectl -n dev apply -f ."
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
    stage('Mark artifact for staging namespace') {
      steps {
        container('docker'){
          sh "cd config && cat ${env.APP_NAME}.yml | grep image: | sed 's/[ \t]*image:[ \t]*//' > image.txt"
          script {
            TAG_DEV = readFile('config\image.txt')
          }
          sh "echo ${TAG_DEV}"
          sh "docker pull ${TAG_DEV}"
          sh "docker tag ${TAG_DEV} ${TAG_STAGING}"
          sh "docker push ${TAG_STAGING}"
        }
      }
    }
    /*
    stage('Commit Configuration change') {
      steps {
        container('git') {
          withCredentials([usernamePassword(credentialsId: 'git-credentials-acm', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
            sh "rm -rf config"
            sh "git config --global user.email ${env.GITHUB_USER_EMAIL}"
            sh "git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/config"
            sh "cd config && git checkout staging"
            // sh "cd config && git checkout -b pr/${env.PR_BRANCH}" 
            sh "cd config && sed -i 's~image: .* #image-green~image: ${env.TAG_STAGING} #image-green~' ${env.APP_NAME}.yml"
            sh "cd config && git add ."
            sh "cd config && git commit -am 'updated config for ${env.APP_NAME}'"
            sh "cd config && git push"
            // sh "cd config && git push --set-upstream https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/config pr/${env.PR_BRANCH}" 
          }
        }
      }
    }
    stage('Deploy to staging') {
      steps {
        build job: "${env.GITHUB_ORGANIZATION}/deploy/staging",
          parameters: [
            string(name: 'APP_NAME', value: "${env.APP_NAME}"),
            string(name: 'TAG_STAGING', value: "${env.TAG_STAGING}"),
            string(name: 'VERSION', value: "${env.VERSION}")
          ]
      }
    }
    */
  }
}
