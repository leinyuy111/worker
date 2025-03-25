def registry= "975050242866.dkr.ecr.ca-central-1.amazonaws.com"
def tag = ""
def ms = "worker"
def region = "ca-central-1"

pipeline{
    agent any
    environment {
        ECR_REPO = '975050242866.dkr.ecr.ca-central-1.amazonaws.com/worker'
    }
    stages{
        stage("init"){
            steps{
                git 'https://github.com/leinyuy111/worker.git'
                script{
                    tag = getTag()
                //    ms = getMsName()
                }
            }
        }
        stage("Build Docker image"){
            steps{
                script{
                    sh "docker build -t worker ."
                }
            }
        }

        stage("Login to Ecr"){
            steps{
                script{
                    withAWS(region:"$region",credentials:'aws_creds'){
                        sh "aws ecr get-login-password --region ca-central-1 | docker login --username AWS --password-stdin $ECR_REPO
                        sh "docker tag worker:latest $ECR_REPO:$tag"
                        sh "docker push $ECR_REPO:$tag"
                    }
                }
            }
        }

        stage("Docker push"){
            steps{
                script{
                    withAWS(region:"$region",credentials:'aws_creds'){
                        sh "docker push ${registry}/${ms}:${tag}"
                    }
                }
            }
        }

        stage("Deploy to Dev"){
            when{branch 'develop'}
            steps{
                script{
                    withAWS(region:"$region",credentials:'aws_creds'){
                        sh "aws eks update-kubeconfig --name worker-dev"
                        sh "kubectl set image deploy/result result=${tag} -n worker"
                        sh "kubectl rollout restart deploy/result -n worker"
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
 version = "1.3.0"
 print "version: ${version}"

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
