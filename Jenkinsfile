pipeline {
    agent any
    
    tools {
        maven 'Maven-3.9.0'  // Tên Maven đã cấu hình trong Jenkins
        jdk 'JDK-17'         // Tên JDK đã cấu hình trong Jenkins
    }
    
    stage('Test') {
        steps {
            script {
                // Cách 1: Lấy danh sách service từ cấu trúc thư mục
                def allServices = sh(
                    script: """ls -d */ | grep -v '^\.' | sed 's/\\/$//' | grep -E '(-service$|bff$|webhook$)'""",
                    returnStdout: true
                ).trim().split('\n')
                
                // Cách 2: Lấy các file đã thay đổi
                def changedFiles = sh(
                    script: "git diff --name-only HEAD^ HEAD",
                    returnStdout: true
                ).trim().split('\n')
                
                // Tìm service bị thay đổi
                def changedServices = []
                for (file in changedFiles) {
                    for (service in allServices) {
                        if (file.startsWith(service + '/')) {
                            changedServices.add(service)
                            break
                        }
                    }
                }
                
                // Loại bỏ trùng lặp
                changedServices = changedServices.unique()
                
                if (changedServices.isEmpty()) {
                    echo "No service changed, skipping tests"
                    return
                }
                
                echo "Changed services: ${changedServices}"
                
                // Chạy test song song cho nhiều service
                def parallelStages = [:]
                for (service in changedServices) {
                    parallelStages[service] = {
                        dir(service) {
                            sh 'mvn clean test'
                        }
                    }
                }
                parallel parallelStages
            }
        }
        
        post {
            always {
                // Upload tất cả test reports từ mọi service
                junit '**/target/surefire-reports/*.xml'
                publishCoverage adapters: [
                    jacocoAdapter('**/target/site/jacoco/jacoco.xml')
                ]
            }
        }
    }        
        // ========== PHASE 2: BUILD ==========
        stage('Build') {
            steps {
                script {
                    def changedServices = sh(
                        script: "git diff --name-only HEAD^ HEAD | cut -d'/' -f1 | sort -u",
                        returnStdout: true
                    ).trim()
                    
                    echo "Building services: ${changedServices}"
                    
                    // Build từng service bị thay đổi
                    if (changedServices.contains('media-service')) {
                        dir('media-service') {
                            sh 'mvn clean compile'
                        }
                    }
                    
                    if (changedServices.contains('product-service')) {
                        dir('product-service') {
                            sh 'mvn clean compile'
                        }
                    }
                    
                    if (changedServices.contains('cart-service')) {
                        dir('cart-service') {
                            sh 'mvn clean compile'
                        }
                    }
                }
            }
        }
    }
    
    post {
        always {
            // Dọn dẹp workspace sau khi pipeline kết thúc
            cleanWs()
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
