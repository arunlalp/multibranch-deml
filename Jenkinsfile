@Library('jenkins-shared-library@develop') _


pipeline {
  agent any

  stages {
    stage('Call jenkins-agent pipeline') {
      when {
        expression { return env.GIT_BRANCH == 'develop' }
      }
      steps {
        script {
          // Change the working directory to the jenkins-agent directory
          dir('infra/jenkins-agent') {
            // Execute the jenkins-agent Jenkinsfile
            load('jenkins-agent.Jenkinsfile')
          }
        }
      }
    }

    stage('Call jenkins-controller pipeline') {
      when {
        expression { return env.GIT_BRANCH == 'develop' }
      }
      steps {
        script {
          // Change the working directory to the jenkins-controller directory
          dir('infra/jenkins-controller') {
            // Execute the jenkins-controller Jenkinsfile
            load('jenkins-controller.Jenkinsfile')
          }
        }
      }
    }
  }
}
