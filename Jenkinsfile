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
                sh 'mvn --version'
                sh 'java -version'
                sh 'mvn clean package -Dmaven.test.failure.ignore=true'
            }
        }

        stage('Packaging/Pushing image') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh 'docker build -t huyqn/springboot .'
                    sh 'docker push huyqn/springboot'
                }
            }
        }

        stage('Deploy MySQL to DEV') {
    steps {
        echo 'Deploying and cleaning'
        sh 'docker image pull mysql:8.0'
        sh 'docker network create dev || echo "this network exists"'
        sh 'echo y | docker container prune'
        
        // Sử dụng withEnv để truyền mật khẩu một cách an toàn
        withEnv(["MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW}"]) {
            sh "docker run --name huyqn-mysql --rm --network dev -v huyqn-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD -e MYSQL_DATABASE=mydb -d mysql:8.0"
        }

        // Đợi 20 giây
        sh "timeout 20 bash -c 'while ! docker exec huyqn-mysql mysqladmin ping -h localhost; do sleep 1; done'"
        
        // Thực thi script
        sh "docker exec -i huyqn-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script.sql"
    }
}

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull huyqn/springboot'
                sh 'docker container stop huyqn-springboot || echo "this container does not exist" '
                sh 'docker network create dev || echo "this network exists"'
                sh 'echo y | docker container prune '
                sh 'docker container run -d --rm --name huyqn-springboot -p 8081:8080 --network dev huyqn/springboot'
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
