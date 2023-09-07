
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
                echo 'Starting to build docker image DB'
                script {
                        dockerImageBack = docker.build("devops-back:$BUILD_NUMBER","-f ${env.WORKSPACE}/devops-back/Dockerfile.api .")
                        dockerImageFront = docker.build("devops-front:$BUILD_NUMBER","-f ${env.WORKSPACE}/devops-front/Dockerfile.client .")
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
                        sh "docker rmi $registry:$BUILD_NUMBER"
                    }  else {
                        /*bat 'docker system prune -af --volumes'*/
                        bat "docker rmi $registry:$BUILD_NUMBER"
                    }
                }
            }
        }        
    }
}