pipeline {
    agent any
    tools{
        maven 'M2_HOME'
    }
    environment {
        registry = '752710401115.dkr.ecr.ap-south-1.amazonaws.com/geolocation_ecr_rep'
        dockerimage = '' 
    }
    stages {
        stage('Checkout'){
            steps{
                git branch: 'main', url: 'https://github.com/tchakouger/Geolocation.git'
            }
        }
        stage('Code Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        // Building Docker images
        stage('Building image') {
            steps{
                script {
                    dockerImage = docker.build registry
                }
            }
        }
        // Uploading Docker images into AWS ECR
        stage('Pushing to ECR') {
            steps{
                script {
                    sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 752710401115.dkr.ecr.ap-south-1.amazonaws.com'
                    sh 'docker push 752710401115.dkr.ecr.ap-south-1.amazonaws.com/geolocation_ecr_rep:latest'
                }
            }
        }
        //deploy the image that is in ECR to our EKS cluster
        stage ("Kube Deploy") {
            steps {
                withKubeConfig([credentialsId: 'eks_credential', serverUrl: '']) {
                 sh "kubectl apply -f eks_deploy_from_ecr.yaml"
                }
            }
        }
    }
}