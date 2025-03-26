def registry = "975050242866.dkr.ecr.ca-central-1.amazonaws.com"
def tag = ""
def ms = "vote"
def region = "ca-central-1"
def sonar_url = "http://3.96.168.151:9000"

pipeline {
    agent any
    environment {
        SONARQUBE_URL = sonar_url
    }
    stages {
        stage("Init") {
            steps {
                script {
                    tag = getTag()
                    // ms = getMsName()
                }
            }
        }
        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        sh "mvn clean verify sonar:sonar -Dsonar.projectKey=${ms} -Dsonar.host.url=$SONARQUBE_URL"
                    }
                }
            }
        }

        stage("Build Docker Image") {
            steps {
                script {
                    sh "docker build . -t ${registry}/${ms}:${tag}"
                }
            }
        }

        stage("Login to ECR") {
            steps {
                script {
                    sh "aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${registry}"
                }
            }
        }

        stage("Docker Push") {
            steps {
                script {
                    sh "docker push ${registry}/${ms}:${tag}"
                }
            }
        }

        stage("Deploy to Dev") {
            when { branch 'develop' }
            steps {
                script {
                    withAWS(region: region, credentials: 'aws_creds') {
                        sh "aws eks update-kubeconfig --name vote-dev"
                        sh "kubectl set image deploy/result result=${registry}/${ms}:${tag} -n vote"
                        sh "kubectl rollout restart deploy/result -n vote"
                    }
                }
            }
        }
    }
}

def getTag() {
    sh "ls -l"
    version = "1.0.0"
    echo "version: ${version}"

    def tag = ""
    if (env.BRANCH_NAME == "main") {
        tag = version
    } else if (env.BRANCH_NAME == "develop") {
        tag = "${version}-develop"
    } else {
        tag = "${version}-${env.BRANCH_NAME}"
    }
    return tag
}
