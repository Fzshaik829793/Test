pipeline {
    agent any
    stages {
        stage('Build Maven') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Fzshaik829793/Test']]])
                sh '/var/lib/jenkins/maven/bin/mvn clean install'

            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def gitCommitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    def dockerImageName = "fzshaik8297:${gitCommitId}"
                    sh "sudo docker build -t ${dockerImageName} ."
                    env.DOCKER_IMAGE_NAME = dockerImageName
                }
            }
        }
        stage('Push Image to Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'fayaz-dockerhub-id', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                        sh "docker push ${env.DOCKER_IMAGE_NAME}"
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def kubeconfigPath = '/root/kubeconfig/config'
                    withCredentials([file(credentialsId: 'fayaz-kube-id', variable: 'kubeconfigPath')]) {
                        sh "helm upgrade --install fayaz-app ./fayaz --set image.repository=${env.DOCKER_IMAGE_NAME}"
                    }
                }
            }
        }
    }
}
