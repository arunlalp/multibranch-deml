@Library('jenkins-shared-library@main') _

def ECR_IMAGE = '078851451276.dkr.ecr.us-west-2.amazonaws.com/arun_private_repo:latest'
def ACTION_PARAM = 'apply'
def GIT_URL = 'https://github.com/pet-clinic-project/terraform.git'
def PROJECT_DIR = 'infra/jenkins-agent'
def TF_VARS_FILE = 'jenkins-agent.tfvars'
def PLAN_FILE = 'tfplan.binary'
def PLAN_JSON_FILE = 'tfplan.json'
def TFVARS_FILE_PATH = '../../vars/infra/dev/jenkins-agent.tfvars'
def CUSTOM_POLICY = 'CUSTOM_AWS_*'
def NOTIFICATION_EMAIL = 'arunsample555@gmail.com'     

pipeline {
    agent {
        docker {
            image ECR_IMAGE
            args '-v /var/run/docker.sock:/var/run/docker.sock --privileged'
            reuseNode true
        }
    }
    // options {
    //     // This is required if you want to clean before build
    //     skipDefaultCheckout(true)
    // }

    parameters {
        string(name: 'action', defaultValue: ACTION_PARAM, description: 'Terraform Action (apply, destroy, etc.)')
    }

    stages {
        // stage('Checkout') {
            // steps {
            //     checkout([
            //         $class: 'GitSCM',
            //         branches: [[name: '*/develop']],
            //         userRemoteConfigs: [[url: GIT_URL]]
            //     ])
            // }
        // }

        stage('Terraform Init') {
            steps {
                // cleanWs()
                script {
                    terraformInit(
                        projectDirectory: PROJECT_DIR,
                        tfstateFile: "jenkins_agent.tfstate",
                        tfvarsFile: TF_VARS_FILE
                    )
                }
            }
        }

        stage('Terraform Validate') {
            steps {
                script {
                    terraformValidate(projectDirectory: PROJECT_DIR)
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                script {
                    terraformPlan(
                        projectDirectory: PROJECT_DIR,
                        variableFile: TF_VARS_FILE,
                        planFile: PLAN_FILE,
                        redirectPlanFile: PLAN_JSON_FILE
                    )
                }
            }
        }

        stage('Terraform Lint') {
            steps {
                script {
                    tfLint(
                        projectDirectory: PROJECT_DIR,
                        tfvarsFile: TFVARS_FILE_PATH
                    )
                }
            }
        }

        // stage('Checkov Scan') {
        //     steps {
        //         script {
        //             checkovScan(
        //                 projectDirectory: PROJECT_DIR,
        //                 planFileJson: PLAN_JSON_FILE,
        //                 customPolicy: CUSTOM_POLICY
        //             )
        //         }
        //     }
        // }

        stage('Terraform Apply') {
            when {
                expression {
                    return (env.BRANCH_NAME == 'main' && params.action == 'apply')
                }
            }      
            steps {
                script {
                    terraformApply(
                        projectDirectory: PROJECT_DIR,
                        variableFile: TF_VARS_FILE
                    )
                }
            }
        }

        stage('Terraform Destroy') {
            when {
                expression {
                    return (env.BRANCH_NAME == 'main' && params.action == 'destroy')
                }
            }
            steps {
                script {
                    terraformDestroy(
                        projectDirectory: PROJECT_DIR,
                        tfstateFile: "jenkins_agent.tfstate",
                        tfvarsFile: TF_VARS_FILE
                    )
                }
            }
        }
    }

    post {
        success {
            script {
                emailNotification.sendEmailNotification('success', NOTIFICATION_EMAIL)
            }
        }
        failure {
            script {
                emailNotification.sendEmailNotification('failure', NOTIFICATION_EMAIL)
            }
        }
    }
}
