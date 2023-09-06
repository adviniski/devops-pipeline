# devops-pipeline

https://blog.devgenius.io/implement-ci-cd-using-jenkins-for-python-application-91a3bcf7d91
docker build -f Dockerfile.api -t react-flask-app-api .
docker build -f Dockerfile.client -t react-flask-app-client .
docker-compose -f docker-compose.yml up -d --build
docker container rm devops-app-frontend-1 devops-app-backend-1 devops-app-db-1