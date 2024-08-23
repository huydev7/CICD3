pipeline {

    agent any

    tools { 
        maven 'my-maven' 
    }
    
    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql-root-login')  // Sử dụng biến môi trường an toàn
    }

    stages {

        stage('Check Docker') {
            steps {
                sh 'docker --version'  // Kiểm tra Docker có sẵn
            }
        }

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
                    sh 'docker build -t huyqn/springboot .'  // Đảm bảo build thành công image
                    sh 'docker push huyqn/springboot'  // Push image lên Docker Hub
                }
            }
        }

        stage('Deploy MySQL to DEV') {
            steps {
                withCredentials([string(credentialsId: 'mysql-root-login', variable: 'MYSQL_ROOT_LOGIN_PSW')]) {
                    echo 'Deploying and cleaning MySQL'
                    sh 'docker image pull mysql:8.0'
                    sh 'docker network create dev || echo "this network exists"'
                    sh 'docker container stop khalid-mysql || echo "this container does not exist"'
                    sh 'echo y | docker container prune'
                    sh 'docker volume rm khalid-mysql-data || echo "no volume"'

                    // Chạy MySQL container với các thông số từ biến môi trường
                    sh "docker run --name khalid-mysql --rm --network dev -v khalid-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW} -e MYSQL_DATABASE=mydb -d mysql:8.0"
                    sh 'sleep 20'
                    sh "docker exec -i khalid-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script"
                }
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning Spring Boot'
                sh 'docker image pull huyqn/springboot'  // Đảm bảo pull đúng image đã push
                sh 'docker container stop khalid-springboot || echo "this container does not exist"'
                sh 'docker network create dev || echo "this network exists"'
                sh 'echo y | docker container prune'

                // Chạy Spring Boot container
                sh 'docker container run -d --rm --name khalid-springboot -p 8081:8080 --network dev huyqn/springboot'
            }
        }
 
    }

    post {
        // Clean after build
        always {
            cleanWs()  // Xóa sạch workspace sau khi build hoàn thành
        }
    }
}
