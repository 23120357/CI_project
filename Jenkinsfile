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
                            sh "mvn -pl ${SERVICE} -am clean test jacoco:report"
                        }
                        post {
                            always {
                                junit testResults: "${SERVICE}/**/*-reports/TEST*.xml", allowEmptyResults: true
                                script {
                                    if (fileExists("${SERVICE}/target/jacoco.exec") || fileExists("${SERVICE}/target/site/jacoco/jacoco.xml")) {
                                        jacoco execPattern: "${SERVICE}/target/**/*.exec", classPattern: "${SERVICE}/target/classes", sourcePattern: "${SERVICE}/src/main/java"
                                    } else {
                                        echo "No JaCoCo coverage artifacts found for ${SERVICE}."
                                    }
                                }
                            }
                        }
                    }

                    stage('Phase 2: Build') {
                        steps {
                            sh "mvn package -pl ${SERVICE} -am -DskipTests"

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
