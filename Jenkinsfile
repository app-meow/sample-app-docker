pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'registry-acc-robot1' // ID của credential Docker trong Jenkins
        DOCKER_IMAGE_NAME = 'registry.gama-kltn-fe.online/meow-app/sample-app-docker'
    }

    stages {
        stage('Checkout') {
            steps {
                // Lấy mã nguồn từ kho GitHub riêng tư
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Lấy ID commit hiện tại
                    def commitId = env.GIT_COMMIT
                    // Xây dựng hình ảnh Docker với commit ID làm tag
                    sh "docker build -t ${DOCKER_IMAGE_NAME}:${commitId} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Đăng nhập vào Docker Registry
                    docker.withRegistry('https://registry.gama-kltn-fe.online', DOCKER_CREDENTIALS_ID) {
                        // Đẩy hình ảnh Docker lên registry
                        sh "docker push ${DOCKER_IMAGE_NAME}:${commitId}"
                    }
                }
            }
        }
    }

    post {
        always {
            // Dọn dẹp không gian làm việc
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
