pipeline {
    agent {
        kubernetes {
            label 'kubeagent'
        }
    }
    

    environment {
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
                    // Xây dựng tên Docker image với commit ID
                    def imageName = "${DOCKER_IMAGE_NAME}:${env.GIT_COMMIT}"
                    
                    // In tên image ra console
                    //echo "Docker Image Name: ${imageName}"
                    
                    // Lưu tên image vào biến môi trường để dùng ở các bước sau
                    env.IMAGE_NAME_FULL = imageName
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
