pipeline {
    agent {
        kubernetes {
            label 'kubeagent'
        }
    }
    

    environment {
        DOCKER_CREDENTIALS_ID = 'registry-acc-robot1' // ID của credential Docker trong Jenkins
        DOCKER_IMAGE_NAME = "${env.REGISTRY_URL}/sample-app-docker"
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
                    echo "${env.REGISTRY_URL}"
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
                    //sleep 600 // seconds
                }
            }
        }
        stage('Check Docker Image') {
            steps {
                container('kaniko') {
                   script {
                    // In trực tiếp biến môi trường bằng lệnh shell
                    sh 'ls /kaniko'
                    }
                }
            }
        }
      
        // stage('Build Docker Image') {
        //     steps {
        //         container('kaniko') {
                    
        //             sh '''
        //             /kaniko/executor \
        //                 --context ./ \
        //                 --dockerfile ./Dockerfile \
        //                 --destination ${IMAGE_NAME_FULL} --skip-tls-verify --insecure\
        //             '''
        //         }
        //     }
        // }

        stage('Update Tag') {
            steps {
            script {
                checkout([$class: 'GitSCM',
                    branches: [[name: 'main' ]],
                    extensions: scm.extensions,
                    userRemoteConfigs: [[
                        url: "${env.MANIFEST_URL_GIT}",
                        credentialsId: 'github-acc'
                    ]]
                ])
                sh 'git checkout main'

                withCredentials([gitUsernamePassword(credentialsId: 'github-acc',gitToolName: 'git-tool')]) {
                    // Lấy email và username từ cấu hình git hiện tại
                    def gitEmail = "admin@mail.com"
                    def gitUsername = "truongnam1"
                
                 // Đọc nội dung của file yaml
                    def yamlFilePath = "overlays/dev/user/user-app/alpine-patch.yaml"
                    def newImageTag = "${env.IMAGE_NAME_FULL}"  // Giả định image mới đã được build và tag

                    // Thay thế 'image: *' bằng image mới nhất
                    sh """
                        sed -i 's|image:.*|image: ${newImageTag}|' ${yamlFilePath}
                    """

                    // Cấu hình thông tin user cho git
                    sh 'git config user.email "${gitEmail}"'
                    sh 'git config user.name "${gitUsername}"'

                    // Add file đã thay đổi và commit
                    sh "git add ${yamlFilePath}"
                    sh 'git commit -m "Update image to ${newImageTag}"'
                    sh 'git push'
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
