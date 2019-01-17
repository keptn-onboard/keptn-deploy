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

pipeline {
  parameters {
    string(name: 'APP_NAME', defaultValue: '', description: 'The name of the service to deploy.', trim: true)
    string(name: 'PULL_REQUEST', defaultValue: '', description: 'The pull request id.', trim: true)
    string(name: 'ENVIRONMENT', defaultValue: '', description: 'The env for which to change the configuration.', trim: true)
  }
  agent {
    label 'kubegit'
  }
  stages {
    stage('Change and commit dev configuration') {
      when {
        expression {
          return env.ENVIRONMENT ==~ 'dev' 
        }
      }
      steps {
        container('git') {
          script {
            ARTEFACT_ID = "sockshop/" + "${env.APP_NAME}"
            BASE_TAG = "${env.DOCKER_REGISTRY_URL}:5000/library/${ARTEFACT_ID}"
            IMAGE_TAG = "${BASE_TAG}:${env.PULL_REQUEST}"
          }
          withCredentials([usernamePassword(credentialsId: 'git-credentials-acm', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
            sh "rm -rf config"
            sh "git config --global user.email ${env.GITHUB_USER_EMAIL}"
            sh "git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/config"
            sh "cd config && git checkout dev"
            /* sh "cd config && git checkout -b pr/${env.PR_BRANCH}" */
            sh "cd config && sed -i 's#image: .*#image: ${IMAGE_TAG}#' ${env.APP_NAME}.yml"
            sh "cd config && git add ."
            sh "cd config && git commit -am '[CI-UPDATECONFIG] Updated config for: ${env.APP_NAME}'"
            sh "cd config && git push"
            /* sh "cd config && git push --set-upstream https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/config pr/${env.PR_BRANCH}" */
          }
        }
      }
    }
    stage('Change and commit staging configuration') {
      when {
        expression {
          return env.ENVIRONMENT ==~ 'staging'
        }
      }
      steps {
        container('git') {
          script {
            ARTEFACT_ID = "sockshop/" + "${env.APP_NAME}"
            BASE_TAG = "${env.DOCKER_REGISTRY_URL}:5000/library/${ARTEFACT_ID}"
            IMAGE_TAG = "${BASE_TAG}:${env.PULL_REQUEST}"
          }
          withCredentials([usernamePassword(credentialsId: 'git-credentials-acm', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
            sh "rm -rf config"
            sh "git config --global user.email ${env.GITHUB_USER_EMAIL}"
            sh "git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/config"
            sh "cd config && git checkout ${env.ENVIRONMENT}"
            // sh "cd config && git checkout -b pr/${env.PR_BRANCH}" 
            sh "cd config && sed -i 's~image: .* #image-green~image: ${IMAGE_TAG} #image-green~' ${env.APP_NAME}.yml"
            sh "cd config && git add ."
            sh "cd config && git commit -am '[CI-UPDATECONFIG] Updated config for: ${env.APP_NAME}'"
            sh "cd config && git push"
            // sh "cd config && git push --set-upstream https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/config pr/${env.PR_BRANCH}"  
          }
        }
      }
    }
    stage('Change and commit production configuration') {
      when {
        expression {
          return env.ENVIRONMENT ==~ 'production'
        }
      }
      steps {
        container('git') {
          script {
            ARTEFACT_ID = "sockshop/" + "${env.APP_NAME}"
            BASE_TAG = "${env.DOCKER_REGISTRY_URL}:5000/library/${ARTEFACT_ID}"
            IMAGE_TAG = "${BASE_TAG}:${env.PULL_REQUEST}"
          }
          withCredentials([usernamePassword(credentialsId: 'git-credentials-acm', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
            sh "rm -rf config"
            sh "git config --global user.email ${env.GITHUB_USER_EMAIL}"
            sh "git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/config"
            sh "cd config && git checkout ${env.ENVIRONMENT}"
            // sh "cd config && git checkout -b pr/${env.PR_BRANCH}" 
            sh "cd config && sed -i 's~image: .* #image-green~image: ${IMAGE_TAG} #image-green~' ${env.APP_NAME}.yml"
            sh "cd config && git add ."
            sh "cd config && git commit -am '[CI-UPDATECONFIG] Updated config for: ${env.APP_NAME}'"
            sh "cd config && git push"
            // sh "cd config && git push --set-upstream https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/config pr/${env.PR_BRANCH}"  
          }
        }
      }
    }
  }
}
