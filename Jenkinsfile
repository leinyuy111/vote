def registry= "975050242866.dkr.ecr.ca-central-1.amazonaws.com"
def tag = ""
def ms = "vote"
def region = "ca-central-1"

pipeline{
    agent any
    stages{
        stage("init"){
            steps{
                script{
                    tag = getTag()
                  //  ms = getMsName()
                }
            }
        }
        stage("Build Docker image"){
            steps{
                script{
                    sh "docker build . -t ${registry}/${ms}:${tag}"
                }
            }
        }

        stage("Login to Ecr"){
            steps{
                script{
                        sh "aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${registry}"
                    }
                }
            }

        stage("Docker push"){
            steps{
                script{
                        sh "docker push ${registry}/${ms}:${tag}"
                    }
                }
            }

        stage("Deploy to Dev"){
            when {branch 'dev'}
            steps{
                script{
                    withAWS(region: region, credentials:'aws_creds'){
                        sh "aws eks update-kubeconfig --name vote-dev"
                        sh "kubectl set image deploy/result result=${registry}/${ms}:${tag} -n vote "
                        sh "kubectl rollout restart deploy/result -n vote"
                    }
                }
            }
        }
    }
}

def getMsName(){
    print env.JOB_NAME
    return env.JOB_NAME.split("/")[0]
}

def getTag(){
 sh "ls -l"
 version = "1.0.0"
 echo "version: ${version}"

 def tag = ""
  if (env.BRANCH_NAME == "main"){
    tag = version
  } else if(env.BRANCH_NAME == "develop"){
    tag = "${version}-develop"
  } else {
    tag = "${version}-${env.BRANCH_NAME}"
  }
return tag
}
