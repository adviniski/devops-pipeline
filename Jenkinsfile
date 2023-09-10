pipeline {
    environment {
        registry = "adviniski/devops-course"
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
                    dockerImageBack = docker.build("devops-back-image"," -f devops-back/Dockerfile.api .")
                    dockerImageFront = docker.build("devops-front-image"," -f devops-front/Dockerfile.client .")
                }
            }
        }
        //stage('Login') {
        //    steps {
        //        bat 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
        //    }
        //}
        stage('Deploy back image') {
            steps{
                script {
                    docker.withRegistry(registry, registryCredential ) {
                        dockerImageBack.push()
                    }
                }
            }
        }
        stage('Deploy front image') {
            steps{
                script {
                    docker.withRegistry(registry, registryCredential ) {
                        dockerImageFront.push()
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
                        bat 'docker rmi devops-front-image'
                    }
                }
            }
        }     
    }
}