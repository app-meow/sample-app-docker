pipeline {
    agent {
        kubernetes {
            label 'kubeagent'
        }
    }

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
        stage('Check Docker Image') {
            steps {
                container('kaniko') {
                    sh '''
                    ls
                    '''
                }
            }
        }

        
        stage('Build Docker Image') {
            steps {
                container('kaniko') {
                    sh '''
                    /kaniko/executor \
                        --context ./ \
                        --dockerfile ./Dockerfile \
                        --destination ${DOCKER_IMAGE_NAME}:${commitId}
                        --insecure
                    '''
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
