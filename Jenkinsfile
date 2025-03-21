def registry = "940090592876.dkr.ecr.ca-central-1.amazonaws.com"
def tag = ""
def ms = "worker"
def region = "ca-central-1"

pipeline {
    agent any
    stages {
        stage("Init") {
            steps {
                script {
                    tag = getTag()
                    ms = getMsName()
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
                    withAWS(region: region, credentials: 'aws_creds') {
                        sh "aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${registry}"
                    }
                }
            }
        }
        stage("Docker Push") {
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
    if (env.JOB_NAME) {
        return env.JOB_NAME.split("/")[0]
    } else {
        return "default-service"
    }
}

def getTag() {
    def version = "1.3.0"
    print "version: ${version}"

    if (env.BRANCH_NAME == "main") {
        return version
    } else if (env.BRANCH_NAME == "develop") {
        return "${version}-develop"
    } else {
        return "${version}-${env.BRANCH_NAME}"
    }
}
