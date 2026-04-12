// pipeline {
//     agent any

//     tools {
//         maven 'Maven-3.9.0'
//         jdk 'JDK-21'
//     }

//     stages {
//         stage('Run Unit Test') {
//             matrix {
//                 axes {
//                     axis {
//                         name 'SERVICE'
//                         values 'cart', 'customer', 'inventory', 'location', 'media', 'order', 'payment', 'payment-paypal', 'product', 'promotion', 'rating', 'tax', 'search', 'webhook'
//                     }
//                 }

//                 // Yêu cầu 6: Chỉ chạy service nào có file thay đổi
//                 when {
//                     anyOf {
//                         changeset "${SERVICE}/**"
//                         changeset 'pom.xml'
//                     }
//                 }

//                 stages {
//                     stage('Test & Coverage') {
//                         steps {
//                             // 1. Chạy Test an toàn từ gốc (Không còn lỗi thiếu common-library)
//                             echo "=== ĐANG CHẠY TEST CHO SERVICE: ${SERVICE} ==="
//                             sh "mvn clean verify -pl ${SERVICE} -am -DskipITs"

//                             // 2. Thuật toán trích xuất % Độ phủ code in ra Log
//                             script {
//                                 def reportPath = "${SERVICE}/target/site/jacoco/index.html"
//                                 if (fileExists(reportPath)) {
//                                     def coverageReport = readFile(file: reportPath) 
//                                     // Dùng Regex tìm phần tfoot (chứa kết quả tổng)
//                                     def matcher = coverageReport =~ /<tfoot>(.*?)<\/tfoot>/
//                                     if (matcher.find()) {
//                                         def coverage = matcher[0][1]
//                                         // Dùng Regex tìm thẻ td chứa số %
//                                         def instructionMatcher = coverage =~ /<td class="ctr2">(.*?)%<\/td>/
//                                         if (instructionMatcher.find()) {
//                                             def coveragePercentage = instructionMatcher[0][1]
//                                             echo "====================================================="
//                                             echo " ĐỘ PHỦ CODE CỦA [${SERVICE.toUpperCase()}] ĐẠT: ${coveragePercentage}%"
//                                             echo "====================================================="
//                                         }
//                                     }
//                                 } else {
//                                     echo " Không tìm thấy file report HTML để trích xuất %."
//                                 }
//                             }
//                         }
//                         post {
//                             always {
//                                 // 1. Upload Pass/Fail (Yêu cầu 5)
//                                 junit testResults: "${SERVICE}/**/*-reports/TEST*.xml", allowEmptyResults: true
                                
//                                 // 2. Upload HTML Report lên giao diện Jenkins
//                                 publishHTML(target: [
//                                     allowMissing: true, 
//                                     alwaysLinkToLastBuild: true,
//                                     keepAll: true,
//                                     reportDir: "${SERVICE}/target/site/jacoco",
//                                     reportFiles: 'index.html',
//                                     reportName: "HTML Report - ${SERVICE.toUpperCase()}"
//                                 ])
//                             }
//                             success {
//                                 // 3. Vẽ biểu đồ Coverage UI (Yêu cầu 5)
//                                 jacoco execPattern: "${SERVICE}/target/**/*.exec", classPattern: "${SERVICE}/target/classes", sourcePattern: "${SERVICE}/src/main/java"
//                             }
//                         }
//                     }
//                 }
//             }
//         }

//         stage('Build') {
//             matrix {
//                 axes {
//                     axis {
//                         name 'SERVICE'
//                         values 'cart', 'customer', 'inventory', 'location', 'media', 'order', 'payment', 'payment-paypal', 'product', 'promotion', 'rating', 'tax', 'search', 'webhook'
//                     }
//                 }

//                 // Chỉ chạy service nào có file thay đổi
//                 when {
//                     anyOf {
//                         changeset "${SERVICE}/**"
//                         changeset 'pom.xml'
//                     }
//                 }

//                 stages {
//                     stage('Build Docker') {
//                         steps {
//                             echo "=== 📦 ĐANG BUILD JAR & IMAGE CHO SERVICE: ${SERVICE} ==="
//                             // 1. Đóng gói file .jar
//                             sh "mvn package -pl ${SERVICE} -am -DskipTests"
                            
//                             // 2. Build Docker 
//                             dir("${SERVICE}") {
//                                 sh "docker build -t yas-${SERVICE}:latest ."
//                             }
//                         }
//                         // Thêm khối post này để đẩy file lên Artifact của Jenkins
//                         post {
//                             success {
//                                 echo "📤 Đang lưu trữ file JAR của ${SERVICE} vào Artifacts..."
//                                 // Thu thập tất cả file .jar trong thư mục target của service đó
//                                 archiveArtifacts artifacts: "${SERVICE}/target/*.jar", fingerprint: true
//                             }
//                         }
//                     }
//                 }
//             }
//         }
//     }
// }
pipeline {
    agent any
    
    tools {
        maven 'Maven3' 
        jdk 'Java21'   
    }

    // Ép Java21
    environment {
        PATH_TO_JAVA = tool name: 'Java21', type: 'jdk'
        JAVA_HOME = "${PATH_TO_JAVA}"
        PATH = "${PATH_TO_JAVA}/bin:${env.PATH}"
    }

    stages {
        stage('Checkout') {
            steps {
                // Lấy code từ GitHub về
                checkout scm
            }
        }

        stage('Test & Coverage') {
            steps {
                echo 'Đang kiểm tra phiên bản Java...'
                sh 'java -version'
                
                echo 'Đang chạy Unit Test và tạo report Coverage cho toàn bộ dự án...'
                sh 'mvn clean test jacoco:report'
            }
        }

        stage('Build') {
            steps {
                echo 'Đang đóng gói toàn bộ ứng dụng (Bỏ qua test vì đã chạy ở stage trước)...'
                sh 'mvn package -DskipTests'
            }
        }
    }

//Lấy báo cáo
    post {
        always {
            echo 'Pipeline hoàn thành (Dù Pass hay Fail). Đang kéo báo cáo Test và Coverage...'
            
            
            junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
            
           
            jacoco execPattern: '**/target/jacoco.exec',
                   classPattern: '**/target/classes',
                   sourcePattern: '**/src/main/java'
        }
    }
}   