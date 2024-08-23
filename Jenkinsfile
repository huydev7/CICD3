pipeline {
    agent any

    tools { 
        maven 'my-maven' 
    }
    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql-root-login')
    }
    stages {

        stage('Build with Maven') {
            steps {
                bat 'mvn --version'
                bat 'java -version'
                bat 'mvn clean package -Dmaven.test.failure.ignore=true'
            }
        }

        stage('Packaging/Pushing image') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    bat 'docker build -t huyqn/springboot .'
                    bat 'docker push huyqn/springboot'
                }
            }
        }

        stage('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                bat 'docker image pull mysql:8.0'
                bat 'docker network create dev || echo "this network exists"'
                bat 'docker container stop khalid-mysql || echo "this container does not exist" '
                bat 'echo y | docker container prune '
                bat 'docker volume rm khalid-mysql-data || echo "no volume"'
                bat "docker run --name khalid-mysql --rm --network dev -v khalid-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW} -e MYSQL_DATABASE=mydb -d mysql:8.0"
                bat 'timeout /t 20'
                bat "docker exec -i khalid-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script"
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                bat 'docker image pull huyqn/springboot'
                bat 'docker container stop khalid-springboot || echo "this container does not exist" '
                bat 'docker network create dev || echo "this network exists"'
                bat 'echo y | docker container prune '
                bat 'docker container run -d --rm --name khalid-springboot -p 8081:8080 --network dev huyqn/springboot'
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
