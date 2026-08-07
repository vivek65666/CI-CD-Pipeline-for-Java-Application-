pipeline {
    agent any
    
    tools {
        jdk 'JDK-17'
        maven 'Maven-3.9'
    }
    
    environment {
        SCANNER_HOME = tool 'SonarQubeScanner'
        DOCKER_CREDS = 'docker-hub-credentials-id' // Replace with your Jenkins credential ID
        NEXUS_CREDS = 'nexus-credentials-id'     // Replace with your Jenkins credential ID
        IMAGE_NAME = 'vivek65666/spring-boot-app'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('1. Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/vivek65666/CI-CD-Pipeline-for-Java-Application-.git'
            }
        }
        
        stage('2. Build & Unit Test') {
            steps {
                sh 'mvn clean test'
            }
            post {
                success {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('3. Code Quality Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    sh "${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectKey=spring-boot-app \
                    -Dsonar.sources=src/main/java \
                    -Dsonar.tests=src/test/java \
                    -Dsonar.java.binaries=target/classes"
                }
            }
        }
        
        stage('4. Publish Artifact to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${NEXUS_CREDS}", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASSWORD')]) {
                    sh 'mvn deploy -DskipTests'
                }
            }
        }
        
        stage('5. Build & Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDS}") {
                        def customImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                        customImage.push()
                        customImage.push('latest')
                    }
                }
            }
        }
        
        stage('6. Deploy to Kubernetes') {
            steps {
                script {
                    sh "sed -i 's|IMAGE_TAG|${IMAGE_TAG}|g' k8s/deployment.yaml"
                    kubernetesDeploy(configs: 'k8s/deployment.yaml', kubeconfigId: 'k8s-kubeconfig-id')
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}