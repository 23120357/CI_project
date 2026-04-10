pipeline {
    agent any

    tools {
        maven 'Maven3'
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
                            dir("${SERVICE}") {
                                sh 'mvn clean verify'
                            }
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
                            dir("${SERVICE}") {
                                sh 'mvn package -DskipTests'
                                sh "docker build -t yas-${SERVICE}:latest ."
                            }
                        }
                    }
                }
            }
        }
    }
}
