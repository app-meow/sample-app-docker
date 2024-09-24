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
    stage('get commit Id') {
            steps {
                script {
                    // Lấy commit ID và lưu vào biến
                    def commitId = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()

                    // In commit ID ra console
                    echo "Commit ID: ${commitId}"

                    // Xây dựng tên Docker image với commit ID
                    def imageName = "${DOCKER_IMAGE_NAME}:${commitId}"
                    
                    // In tên image ra console
                    echo "Docker Image Name: ${imageName}"

                    // Lưu tên image vào biến môi trường để dùng ở các bước sau
                    env.IMAGE_NAME_FULL = imageName
                }
            }
        }
        stage('Check Docker Image') {
            steps {
                container('kaniko') {
                   script {
                    // In trực tiếp biến môi trường bằng lệnh shell
                    sh 'echo $IMAGE_NAME_FULL'
                    }
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
                        --destination ${IMAGE_NAME_FULL} --skip-tls-verify \
                    '''
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
