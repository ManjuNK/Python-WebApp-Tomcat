pipeline {
    agent any

    environment {
        SCANNER_HOME=tool 'SonarQube'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ManjuNK/Python-WebApp.git'
            }
        }
        
        stage("OWASP Dependency"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage("Trivy FS "){
            steps{
                sh "trivy fs ."
            }
        } 
        
        stage(" Sonarqube Analysis "){
            steps{
                 withSonarQubeEnv('Sonar-Server-9.4') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Python-webapp \
                    -Dsonar.projectKey=Python-webapp '''
                    
                 }
            }
        } 
        
        stage("Docker Build and Tag  "){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "make image"
                   }
                }
            }
        } 
        
         stage("Trivy Image Scan "){
            steps{
                sh "trivy image manjunk/python-webapp "
            }
        } 
        stage("Docker Push"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "make push"
                   }
                }
            }
        } 
        
        stage("Deploy to container"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "docker run -d -p 5000:5000 manjunk/python-webapp:latest "
                   }
                }
            }
        } 
    }
}
