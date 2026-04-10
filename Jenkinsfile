pipeline {
    agent any

    tools {
        maven 'Maven-3.9.0'
        jdk 'JDK-25'
    }

    stages {
        stage('CI Matrix') {
            matrix {
                axes {
                    axis {
                        name 'SERVICE'
                        values 'backoffice-bff', 'backoffice', 'cart', 'customer', 'inventory', 'location', 'media', 'order', 'payment', 'payment-paypal', 'product', 'promotion', 'rating'
                    }
                }

                when {
                    anyOf {
                        changeset "${SERVICE}/**"
                        changeset 'pom.xml'
                    }
                }

                stages {
                    stage('Phase 1: Test') {
                        steps {
                            // SỬA Ở ĐÂY: KHÔNG dùng dir() nữa, chạy Maven từ thư mục gốc với -pl và -am
                            sh "mvn clean verify -pl ${SERVICE} -am"
                        }
                        post {
                            always {
                                junit testResults: "${SERVICE}/**/*-reports/TEST*.xml", allowEmptyResults: true
                            }
                            success {
                                jacoco execPattern: "${SERVICE}/target/**/*.exec", classPattern: "${SERVICE}/target/classes", sourcePattern: "${SERVICE}/src/main/java"
                            }
                        }
                    }

                    stage('Phase 2: Build') {
                        steps {
                            // SỬA Ở ĐÂY: Build file .jar bằng Maven từ thư mục gốc
                            sh "mvn package -pl ${SERVICE} -am -DskipTests"
                            
                            // Chỉ chui vào thư mục service khi cần build Docker Image
                            dir("${SERVICE}") {
                                sh "docker build -t yas-${SERVICE}:latest ."
                            }
                        }
                    }
                }
            }
        }
    }
}