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

        stage('Packaging/Pushing image') { // Đã sửa lỗi chính tả 'imagae' thành 'image'
            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh 'docker build -t huyqn/springboot .'
                    sh 'docker push huyqn/springboot'
                }
            }
        } // Thêm dấu ngoặc nhọn đóng cho stage này
    } // Thêm dấu ngoặc nhọn đóng cho stages

    post {
        // Clean after build
        always {
            cleanWs()
        }
    }
}
