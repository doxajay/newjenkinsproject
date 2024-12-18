pipeline {
    tools {
        maven 'maven'
    }
    environment {
  PATH = "/usr/local/bin:${env.PATH}"
   }

    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/doxajay/newjenkinsproject.git']]])
            }
        }
        stage('Build Jar') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Docker Image Build') {
            steps {
                sh 'docker build -t projectjenkins .'
            }
        }
        stage('Push Docker Image to ECR') {
            steps {
                withAWS(credentials: 'aws-cred', region: 'us-west-2') {
                    sh 'aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 460982569648.dkr.ecr.us-west-2.amazonaws.com'
                    sh 'docker tag projectjenkins:latest 460982569648.dkr.ecr.us-west-2.amazonaws.com/eksfin:latest'
                    sh 'docker push 460982569648.dkr.ecr.us-west-2.amazonaws.com/eksfin:latest'
                }
            }
        }
        stage('Integrate Jenkins with EKS Cluster and Deploy App') {
            steps {
                withAWS(credentials: 'aws-cred', region: 'us-west-2') {
                  script {
                    sh ('aws eks --region us-west-2 update-kubeconfig --name ecrnew')
                    sh '/usr/local/bin/kubectl apply -f eks-deploy-k8s.yaml'
                }
                }
        }
    }
    }
}
