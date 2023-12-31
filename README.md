# devops-pipeline

https://blog.devgenius.io/implement-ci-cd-using-jenkins-for-python-application-91a3bcf7d91

# docker build -f Dockerfile.api -t react-flask-app-api .

# docker build -f Dockerfile.client -t react-flask-app-client .

# docker-compose -f docker-compose.yml up -d --build

# docker-compose -f docker-compose.yml up --build

# docker container rm devops-app-frontend-1 devops-app-backend-1 devops-app-db-1

# docker container stop

# docker ps -a


pipeline {
    environment {
        registry = "adviniski/devops-course" //To push an image to Docker Hub, you must first name your local image using your Docker Hub username and the repository name that you created through Docker Hub on the web.
        registryCredential = 'docker-hub-token'
        githubCredential = 'git-hub-token'
        dockerImage = ''
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
            script {
                dockerImage = docker.build registry + ":$BUILD_NUMBER"
            }
        }
        }
        stage('Deploy production image') {
            steps{
                script {
                    docker.withRegistry( '', registryCredential ) {
                        dockerImage.push()
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
    /*post {
        always {
            echo 'The pipeline completed'
            junit allowEmptyResults: true, testResults:'**/test_reports/*.xml'
        }
        success {                   
            echo "Flask Application Up and running!!"
        }
        failure {
            echo 'Build stage failed'
            error('Stopping early…')
        }
    }*/
}