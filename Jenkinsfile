pipeline {
    agent any
    
    tools {
        maven 'Maven-3.9.0'  // Tên Maven đã cấu hình trong Jenkins
        jdk 'JDK-17'         // Tên JDK đã cấu hình trong Jenkins
    }
    
    stages {
        // ========== PHASE 1: TEST ==========
        stage('Test') {
            steps {
                script {
                    // Xác định service đã thay đổi (cho monorepo)
                    def changedServices = sh(
                        script: "git diff --name-only HEAD^ HEAD | cut -d'/' -f1 | sort -u",
                        returnStdout: true
                    ).trim()
                    
                    echo "Changed services: ${changedServices}"
                    
                    // Chạy test cho từng service bị thay đổi
                    if (changedServices.contains('media-service')) {
                        dir('media-service') {
                            sh 'mvn clean test'
                        }
                    }
                    
                    if (changedServices.contains('product-service')) {
                        dir('product-service') {
                            sh 'mvn clean test'
                        }
                    }
                    
                    if (changedServices.contains('cart-service')) {
                        dir('cart-service') {
                            sh 'mvn clean test'
                        }
                    }
                }
            }
            
            // Upload test result và độ phủ code lên Jenkins
            post {
                always {
                    // Upload kết quả test (JUnit XML reports)
                    junit '**/target/surefire-reports/*.xml'
                    
                    // Upload báo cáo độ phủ code (JaCoCo)
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
