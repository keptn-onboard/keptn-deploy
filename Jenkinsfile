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

def TAG_IMAGE = 'UNKNOWN'

pipeline {
  parameters {
    string(name: 'APP_NAME', defaultValue: '', description: 'The name of the service to deploy.', trim: true)
  }
  agent {
    label 'kubegit'
  }
  stages {
    stage('Checkout repo') {
      steps {
        container('git') {
          withCredentials([usernamePassword(credentialsId: 'git-credentials-acm', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
            sh "rm -rf ${env.APP_NAME}"
            sh "git config --global user.email ${env.GITHUB_USER_EMAIL}"
            sh "git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/${env.APP_NAME}"
            script {
              TAG_IMAGE = readFile("${env.APP_NAME}/artifact.id").trim();
            }
          }
        }
      }
    }
    stage('Commit Configuration change') {
      steps {
        container('git') {
          withCredentials([usernamePassword(credentialsId: 'git-credentials-acm', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
            sh "rm -rf config"
            sh "git config --global user.email ${env.GITHUB_USER_EMAIL}"
            sh "git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/config"
            sh "cd config && git checkout dev"
            /* sh "cd config && git checkout -b pr/${env.PR_BRANCH}" */
            sh "cd config && sed -i 's#image: .*#image: ${TAG_IMAGE}#' ${env.APP_NAME}.yml"
            sh "cd config && git add ."
            sh "cd config && git commit -am 'Updated config for ${env.APP_NAME}'"
            sh "cd config && git push"
            /* sh "cd config && git push --set-upstream https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/config pr/${env.PR_BRANCH}" */
          }
        }
      }
    }
  }
}
