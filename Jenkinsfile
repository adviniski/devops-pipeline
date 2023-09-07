
pipeline {
    environment {
        registry = "adviniski/devops-course" //To push an image to Docker Hub, you must first name your local image using your Docker Hub username and the repository name that you created through Docker Hub on the web.
        registryCredential = 'docker-hub-token'
        githubCredential = 'git-hub-token'
        dockerImageBack = ''
        dockerImageFront = ''
    }
    agent any
    stages {
        stage('CheckoutPipeline') {
            steps {
                git branch: "main",
                credentialsId: githubCredential,
                url: 'https://github.com/adviniski/devops-pipeline.git'
            }
        }
        stage('CheckoutBackend') {
            steps {
                dir("devops-back")
                {
                    git branch: "main",
                    credentialsId: githubCredential,
                    url: 'https://github.com/adviniski/devops-back.git'
                }
            }
        }
        stage('CheckoutFront') {
            steps {
                dir("devops-front")
                {
                    git branch: "main",
                    credentialsId: githubCredential,
                    url: 'https://github.com/adviniski/devops-front.git'
                }
            }
        }
        stage('Build Test Project') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'docker-compose -f docker-compose.yml up -d --build'
                    }  else {
                        /*bat 'docker system prune -af --volumes'*/
                        bat 'docker-compose -f docker-compose.yml up -d --build'
                    }
                }      
            }
        }
        stage('Building our image') {
            steps{
                echo 'Starting to build docker images'
                script {
                        dockerImageBack = docker.build registry "devops-back:$BUILD_NUMBER-f /devops-back/Dockerfile.api ."
                        dockerImageFront = docker.build registry "devops-front:$BUILD_NUMBER -f /devops-front/Dockerfile.client ."
                }
            }
        }
        stage('Deploy production image') {
            steps{
                script {
                    docker.withRegistry( '', registryCredential ) {
                        dockerImageBack.push()
                        dockerImageFront.push()
                    }
                }
            }
        }
        stage('Cleaning up') {
            steps{
                script {
                    if (isUnix()) {
                        sh "docker rmi devops-back:$BUILD_NUMBER"
                    }  else {
                        bat "docker rmi devops-front:$BUILD_NUMBER"
                    }
                }
            }
        }        
    }
}