def registry= "940090592876.dkr.ecr.ca-central-1.amazonaws.com"
def tag = ""
def ms = "vote"
def region = "ca-central-1"

pipeline {
    agent any
    stages {
        stage("init") {
            steps {
                script {
                    tag = getTag()
                    ms = getMsName()
                }
            }
        }
        stage("Build Docker image") {
            steps {
                script {
                    sh "docker build . -t ${registry}/${ms}:${tag}"
                }
            }
        }
        stage("Login to ECR") {
            steps {
                script {
                    withAWS(region: region, credentials: 'aws_creds') {
                        sh "aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${registry}"
                    }
                }
            }
        }
        stage("Docker push") {
            steps {
                script {
                    withAWS(region: region, credentials: 'aws_creds') {
                        sh "docker push ${registry}/${ms}:${tag}"
                    }
                }
            }
        }
        stage("Deploy to Dev") {
            when { branch 'develop' }
            steps {
                script {
                    withAWS(region: region, credentials: 'aws_creds') {
                        sh "aws eks update-kubeconfig --name vote-dev"
                        sh "kubectl set image deployment/vote-app vote-app=${registry}/${ms}:${tag} -n vote"
                        sh "kubectl rollout restart deployment/vote-app -n vote"
                    }
                }
            }
        }
    }
}

def getMsName() {
    return env.JOB_NAME ? env.JOB_NAME.split("/")[0] : "default-service"
}

def getTag() {
    sh "ls -l"
    def version = "1.0.0"
    print "version: ${version}"

    def tag = (env.BRANCH_NAME == "main") ? version :
              (env.BRANCH_NAME == "develop") ? "${version}-develop" :
              "${version}-${env.BRANCH_NAME}"

    return tag
}
