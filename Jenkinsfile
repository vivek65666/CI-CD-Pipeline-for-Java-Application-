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
                script {
                    writeFile file: 'settings.xml', text: """<settings>
  <servers>
    <server>
      <id>nexus-credentials-id</id>
      <username>admin</username>
      <password>b91b081d-4ffb-4edf-874d-ac2ac05c2d7f</password>
    </server>
  </servers>
</settings>"""
                    bat 'mvn deploy -DskipTests --settings settings.xml'
                }
            }
        }
        
   stage('5. Build & Push Docker Image') {
    steps {
        script {
            bat '"C:\\Users\\Vivek\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe" login -u vivek65666 -p dckr_pat_wNt9-bxzzrCPNuvjvj_XfdZ3GFk'
            bat '"C:\\Users\\Vivek\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe" build -t vivek65666/spring-boot-app:latest .'
            bat '"C:\\Users\\Vivek\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe" push vivek65666/spring-boot-app:latest'
        }
    }
}
        
        stage('6. Deploy to Kubernetes') {
    steps {
        script {
            // Replace placeholder IMAGE_TAG with the actual build number
            bat "powershell -Command \"(Get-Content k8s/deployment.yaml) -replace 'IMAGE_TAG', '${IMAGE_TAG}' | Set-Content k8s/deployment.yaml\""
            bat 'kubectl apply -f k8s/deployment.yaml'
        }
    }
}
    
    post {
        always {
            cleanWs()
        }
    }
}
