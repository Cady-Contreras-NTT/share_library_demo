// Jenkins Shared Library - Pipeline Completo para DevSecOps
// Ubicación: vars/devSecOpsPipeline.groovy

def call(Map params) {
    pipeline {
        agent any
        environment {
            SONARQUBE_SERVER = credentials('sonarqube-token')
            DOCKER_CREDENTIALS = credentials('docker-hub')
            K8S_CREDENTIALS = credentials('k8s-config')
        }
        stages {
            stage('Checkout') {
                steps {
                    script {
                        checkout scm
                    }
                }
            }
            
            stage('Build') {
                steps {
                    script {
                        sh 'mvn clean package -DskipTests'
                    }
                }
            }
            
            stage('Static Code Analysis') {
                steps {
                    script {
                        sh 'sonar-scanner -Dsonar.projectKey=${JOB_NAME} -Dsonar.host.url=${SONARQUBE_SERVER}'
                    }
                }
            }
            
            stage('Security Scanning') {
                parallel {
                    stage('SAST - Static Analysis') {
                        steps {
                            sh 'snyk test'
                        }
                    }
                    stage('DAST - Dynamic Analysis') {
                        steps {
                            sh 'zap-cli quick-scan http://my-app-url.com'
                        }
                    }
                }
            }
            
            stage('Unit & Integration Tests') {
                steps {
                    script {
                        sh 'mvn test'
                    }
                }
            }
            
            stage('Build Docker Image') {
                steps {
                    script {
                        sh "docker build -t my-app:${env.BUILD_NUMBER} ."
                        sh "docker login -u ${DOCKER_CREDENTIALS_USR} -p ${DOCKER_CREDENTIALS_PSW}"
                        sh "docker push my-app:${env.BUILD_NUMBER}"
                    }
                }
            }
            
            stage('Deploy to Kubernetes') {
                steps {
                    script {
                        sh "kubectl apply -f k8s/deployment.yaml --kubeconfig=${K8S_CREDENTIALS}"
                    }
                }
            }
        }
        post {
            success {
                slackSend channel: '#devops', message: 'Deployment successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}'
            }
            failure {
                slackSend channel: '#devops', message: 'Deployment failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}'
            }
        }
    }
}
