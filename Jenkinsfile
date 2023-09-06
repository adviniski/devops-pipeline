
pipeline {
    environment {
        registry = "adviniski/devops-course" //To push an image to Docker Hub, you must first name your local image using your Docker Hub username and the repository name that you created through Docker Hub on the web.
        registryCredential = 'DOCKERHUB'
        githubCredential = 'GITHUB'
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
        stage('Deploy Project') {
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
    post {
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
      }
}