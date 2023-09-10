pipeline {
    environment {
        registryBack = "adviniski/devops-back"
        registryFront = "adviniski/devops-front"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        registryCredential = 'dockerhub'
        githubCredential = 'github'
        dockerImageBack = ''
        dockerImageFront = ''
    }
    agent any
    stages {
        stage('CheckoutPipeline') {
            steps {
                git branch: 'main',
                credentialsId: githubCredential,
                url: 'https://github.com/adviniski/devops-pipeline.git'
            }
        }
        stage('CheckoutBackend') {
            steps {
                dir('devops-back')
                {
                    git branch: 'main',
                    credentialsId: githubCredential,
                    url: 'https://github.com/adviniski/devops-back.git'
                }
            }
        }
        stage('CheckoutFront') {
            steps {
                dir('devops-front')
                {
                    git branch: 'main',
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
                    dockerImageBack = docker.build("${registryBack}:${env.BUILD_NUMBER}"," -f devops-back/Dockerfile.api .")
                    dockerImageFront = docker.build("${registryFront}:${env.BUILD_NUMBER}"," -f devops-front/Dockerfile.client .")
                }
            }
        }
        stage('Deploy images') {
            steps{
                script {
                    withCredentials([usernamePassword( credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        docker.withRegistry('', 'dockerhub') {
                            bat "docker login -u ${USERNAME} -p ${PASSWORD} --password-stdin"
                            dockerImageBack.push("${env.BUILD_NUMBER}")
                            dockerImageBack.push("latest")
                            dockerImageFront.push("${env.BUILD_NUMBER}")
                            dockerImageFront.push("latest")
                        }
                    }
                }
            }
        }
        
        stage('Cleaning up') {
            steps{
                script {
                    if (isUnix()) {
                        sh 'docker rmi devops-back-image'
                    }  else {
                        bat 'docker rmi devops-back-image'
                    }
                }
            }
        }     
    }
}