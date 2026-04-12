pipeline {
    agent any

    tools {
        maven 'Maven-3.9.0'
        jdk 'JDK-21'
    }

    stages {
        stage('CI Matrix') {
            matrix {
                axes {
                    axis {
                        name 'SERVICE'
                        values 'cart', 'customer', 'inventory', 'location', 'media', 'order', 'payment', 'payment-paypal', 'product', 'promotion', 'rating', 'tax', 'search', 'webhook'
                    }
                }

                // Yêu cầu 6: Chỉ chạy service nào có file thay đổi
                when {
                    anyOf {
                        changeset "${SERVICE}/**"
                        changeset 'pom.xml'
                    }
                }

                stages {
                    stage('Phase 1: Test & Coverage') {
                        steps {
                            // 1. Chạy Test an toàn từ gốc (Không còn lỗi thiếu common-library)
                            echo "=== ĐANG CHẠY TEST CHO SERVICE: ${SERVICE} ==="
                            sh "mvn clean verify -pl ${SERVICE} -am -DskipITs"

                            // 2. Thuật toán trích xuất % Độ phủ code in ra Log
                            script {
                                def reportPath = "${SERVICE}/target/site/jacoco/index.html"
                                if (fileExists(reportPath)) {
                                    def coverageReport = readFile(file: reportPath) 
                                    // Dùng Regex tìm phần tfoot (chứa kết quả tổng)
                                    def matcher = coverageReport =~ /<tfoot>(.*?)<\/tfoot>/
                                    if (matcher.find()) {
                                        def coverage = matcher[0][1]
                                        // Dùng Regex tìm thẻ td chứa số %
                                        def instructionMatcher = coverage =~ /<td class="ctr2">(.*?)%<\/td>/
                                        if (instructionMatcher.find()) {
                                            def coveragePercentage = instructionMatcher[0][1]
                                            echo "====================================================="
                                            echo " ĐỘ PHỦ CODE CỦA [${SERVICE.toUpperCase()}] ĐẠT: ${coveragePercentage}%"
                                            echo "====================================================="
                                        }
                                    }
                                } else {
                                    echo " Không tìm thấy file report HTML để trích xuất %."
                                }
                            }
                        }
                        post {
                            always {
                                // Yêu cầu 5: Upload Pass/Fail
                                junit testResults: "${SERVICE}/**/*-reports/TEST*.xml", allowEmptyResults: true
                            }
                            success {
                                // Yêu cầu 5: Vẽ biểu đồ Coverage UI
                                jacoco execPattern: "${SERVICE}/target/**/*.exec", classPattern: "${SERVICE}/target/classes", sourcePattern: "${SERVICE}/src/main/java"
                            }
                        }
                    }

                    stage('Phase 2: Build Docker') {
                        steps {
                            echo "=== 📦 ĐANG BUILD IMAGE CHO SERVICE: ${SERVICE} ==="
                            // // Đóng gói .jar từ gốc
                            // sh "mvn package -pl ${SERVICE} -am -DskipTests"
                            
                            // // Chui vào thư mục để chạy Dockerfile
                            // dir("${SERVICE}") {
                            //     sh "docker build -t yas-${SERVICE}:latest ."
                            // }
                        }
                    }
                }
            }
        }
    }
}
