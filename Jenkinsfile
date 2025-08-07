
pipeline {
    agent any

    environment {
        SONARQUBE = 'SonarQube'
        NEXUS_URL = 'http://nexus:8081'
        NEXUS_REPO = 'maven-releases'
        ARTIFACT = "app-${new Date().format('yyyyMMddHHmmss')}.jar"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'ganesh.developer', url: 'https://github.com/ganeshsp2296/ci-cd-minikube.git'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }

        stage('Build Artifact') {
            steps {
                sh 'mvn package'
                sh 'cp target/*.jar $ARTIFACT'
            }
        }

        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'app', file: "${ARTIFACT}", type: 'jar']],
                    credentialsId: 'nexus-creds',
                    groupId: 'com.example',
                    nexusUrl: "${NEXUS_URL}",
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    repository: "${NEXUS_REPO}",
                    version: "${new Date().format('yyyyMMddHHmmss')}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t nexus:8081/app:${env.BUILD_NUMBER} ."
            }
        }

        stage('Push Docker Image to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-creds', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh 'docker login nexus:8081 -u $USER -p $PASS'
                    sh "docker push nexus:8081/app:${env.BUILD_NUMBER}"
                }
            }
        }
    }
}
