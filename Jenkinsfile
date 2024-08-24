pipeline {
    agent any

    tools { 
        maven 'my-maven' 
    }
    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql-root-login') // Đảm bảo rằng credential này tồn tại
        MYSQL_ROOT_LOGIN_PSW = credentials('mysql-root-password') // Đảm bảo rằng credential này tồn tại
    }
    stages {

        stage('Build with Maven') {
            steps {
                script {
                    // Chạy lệnh Maven từ thư mục gốc
                    sh 'mvn --version'
                    sh 'java -version'
                    sh 'mvn clean package -Dmaven.test.failure.ignore=true'
                }
            }
        }

        stage('Copy SQL Script') {
            steps {
                script {
                    // Sao chép script.sql vào thư mục workspace
                    sh 'cp path/to/script.sql script/script.sql' // Thay 'path/to/script.sql' bằng đường dẫn thực tế
                }
            }
        }

        stage('Packaging/Pushing image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                        sh 'docker build -t huyqn/springboot .'
                        sh 'docker push huyqn/springboot'
                    }
                }
            }
        }

        stage('Deploy MySQL to DEV') {
            steps {
                script {
                    echo 'Deploying MySQL container'
                    sh 'docker image pull mysql:8.0'
                    sh 'docker network create dev || echo "this network exists"'
                    
                    // Dừng và xóa container nếu nó đang chạy
                    sh 'docker stop huyqn-mysql || echo "Container huyqn-mysql is not running"'
                    sh 'docker rm huyqn-mysql || echo "Container huyqn-mysql does not exist"'
                    
                    // Chạy container MySQL mới
                    withEnv(["MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW}"]) {
                        sh """
                            docker run --name huyqn-mysql --rm --network dev \
                            -v huyqn-mysql-data:/var/lib/mysql \
                            -e MYSQL_ROOT_PASSWORD=\$MYSQL_ROOT_PASSWORD \
                            -e MYSQL_DATABASE=mydb -d mysql:8.0
                        """
                    }

                    // Đợi MySQL khởi động
                    sh "timeout 30 bash -c 'while ! docker exec huyqn-mysql mysqladmin ping -h localhost --silent; do sleep 1; done'"
                    
                    // Thực thi script.sql
                    sh '''
                        if [ -f script/script.sql ]; then
                            echo "Executing script.sql..."
                            docker exec -i huyqn-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script/script.sql
                        else
                            echo "script.sql not found!"
                            exit 1
                        fi
                    '''
                }
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                script {
                    echo 'Deploying Spring Boot application'
                    sh 'docker image pull huyqn/springboot'
                    sh 'docker container stop huyqn-springboot || echo "this container does not exist" '
                    sh 'docker network create dev || echo "this network exists"'
                    sh 'echo y | docker container prune '
                    sh 'docker container run -d --rm --name huyqn-springboot -p 8081:8080 --network dev huyqn/springboot'
                }
            }
        }
    }
    post {
        always {
            script {
                cleanWs()
            }
        }
    }
}
