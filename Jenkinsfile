pipeline {
    agent any
    
    tools {
        jdk 'JDK-17'
        maven 'Maven-3.9'
    }
    
    environment {
        SCANNER_HOME = tool 'SonarQubeScanner'
        DOCKER_CREDS = 'docker-hub-credentials-id'
        NEXUS_CREDS = 'nexus-credentials-id'
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
                bat 'mvn clean test'
            }
        }
        
        stage('3. Code Quality Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    bat "${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectKey=spring-boot-app \
                    -Dsonar.sources=src/main/java \
                    -Dsonar.java.binaries=target/classes"
                }
            }
        }
        
       stage('4. Publish Artifact to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-credentials-id', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASSWORD')]) {
                    script {
                        // Write a temporary Maven settings.xml with the injected credentials
                        writeFile file: 'settings.xml', text: """<settings>
  <servers>
    <server>
      <id>nexus-credentials-id</id>
      <username>\${NEXUS_USER}</username>
      <password>\${NEXUS_PASSWORD}</password>
    </server>
  </servers>
</settings>"""
                        // Run maven deploy using the custom settings file
                        bat 'mvn deploy -DskipTests --settings settings.xml'
                    }
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
                    bat "powershell -Command \"(Get-Content k8s/deployment.yaml) -replace 'IMAGE_TAG', '${IMAGE_TAG}' | Set-Content k8s/deployment.yaml\""
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
