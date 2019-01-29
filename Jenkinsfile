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
def ARTEFACT_ID = 'UNKNOWN'
def BASE_TAG = 'UNKNOWN'

pipeline {
  parameters {
    string(name: 'APP_NAME', defaultValue: '', description: 'The name of the service to deploy.', trim: true)
    string(name: 'IMAGE', defaultValue: '', description: 'The name of the container image to deploy.', trim: true)
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
            ARTEFACT_ID = "${env.GITHUB_ORGANIZATION}/" + "${env.IMAGE}"
            BASE_TAG = "${env.DOCKER_REGISTRY_URL}:5000/library/${ARTEFACT_ID}"
            IMAGE_TAG = "${BASE_TAG}:${env.PULL_REQUEST}"
          }
          withCredentials([usernamePassword(credentialsId: 'git-credentials-acm', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
            sh "rm -rf keptn-config"
            sh "git config --global user.email ${env.GITHUB_USER_EMAIL}"
            sh "git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/keptn-config"
            sh "cd keptn-config && git checkout dev"
            /* sh "cd keptn-config && git checkout -b pr/${env.PR_BRANCH}" */
          }
        }
        container('yq') {
          sh "cd keptn-config && yq w -i helm-chart/values.yaml ${env.APP_NAME}.image.repository ${BASE_TAG}"
          sh "cd keptn-config && yq w -i helm-chart/values.yaml ${env.APP_NAME}.image.tag ${env.PULL_REQUEST}"
        }
        container('git') {
          withCredentials([usernamePassword(credentialsId: 'git-credentials-acm', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {    
            sh "cd keptn-config && git add ."
            sh "cd keptn-config && git commit -am '[CI-UPDATECONFIG] Updated config for: ${env.APP_NAME}'"
            sh "cd keptn-config && git push"
            /* sh "cd keptn-config && git push --set-upstream https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/keptn-config pr/${env.PR_BRANCH}" */
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
            ARTEFACT_ID = "${env.GITHUB_ORGANIZATION}/" + "${env.APP_NAME}"
            BASE_TAG = "${env.DOCKER_REGISTRY_URL}:5000/library/${ARTEFACT_ID}"
            IMAGE_TAG = "${BASE_TAG}:${env.PULL_REQUEST}"
          }
          withCredentials([usernamePassword(credentialsId: 'git-credentials-acm', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
            sh "rm -rf keptn-config"
            sh "git config --global user.email ${env.GITHUB_USER_EMAIL}"
            sh "git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/keptn-config"
            sh "cd keptn-config && git checkout ${env.ENVIRONMENT}"
            // sh "cd keptn-config && git checkout -b pr/${env.PR_BRANCH}"
          }
        }
        container('yq') {
          sh "cd keptn-config && yq w -i helm-chart/values.yaml ${env.APP_NAME}Green.image.repository ${BASE_TAG}"
          sh "cd keptn-config && yq w -i helm-chart/values.yaml ${env.APP_NAME}Green.image.tag ${env.PULL_REQUEST}"
        }
        container('git') {
          withCredentials([usernamePassword(credentialsId: 'git-credentials-acm', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) { 
            sh "cd keptn-config && git add ."
            sh "cd keptn-config && git commit -am '[CI-UPDATECONFIG] Updated config for: ${env.APP_NAME}'"
            sh "cd keptn-config && git push"
            // sh "cd keptn-config && git push --set-upstream https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/keptn-config pr/${env.PR_BRANCH}"  
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
            ARTEFACT_ID = "${env.GITHUB_ORGANIZATION}/" + "${env.APP_NAME}"
            BASE_TAG = "${env.DOCKER_REGISTRY_URL}:5000/library/${ARTEFACT_ID}"
            IMAGE_TAG = "${BASE_TAG}:${env.PULL_REQUEST}"
          }
          withCredentials([usernamePassword(credentialsId: 'git-credentials-acm', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
            sh "rm -rf keptn-config"
            sh "git config --global user.email ${env.GITHUB_USER_EMAIL}"
            sh "git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/keptn-config"
            sh "cd keptn-config && git checkout ${env.ENVIRONMENT}"
            // sh "cd keptn-config && git checkout -b pr/${env.PR_BRANCH}"
          }
        }
        container('yq') {
          sh "cd keptn-config && yq w -i helm-chart/values.yaml ${env.APP_NAME}Green.image.repository ${BASE_TAG}"
          sh "cd keptn-config && yq w -i helm-chart/values.yaml ${env.APP_NAME}Green.image.tag ${env.PULL_REQUEST}"
        }
        container('git') {
          withCredentials([usernamePassword(credentialsId: 'git-credentials-acm', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) { 
            sh "cd keptn-config && git add ."
            sh "cd keptn-config && git commit -am '[CI-UPDATECONFIG] Updated config for: ${env.APP_NAME}'"
            sh "cd keptn-config && git push"
            // sh "cd keptn-config && git push --set-upstream https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/keptn-config pr/${env.PR_BRANCH}"  
          }
        }
      }
    }
  }
}
