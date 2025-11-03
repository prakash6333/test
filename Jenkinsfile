pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SONARQUBE_ENV = 'sonarqube'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/prakash6333/test.git', branch: 'main'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean package'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=test-app -Dsonar.projectName=test-app'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy to Nexus') {
            steps {
                configFileProvider([configFile(fileId: 'maven-settings-id', variable: 'MAVEN_SETTINGS')]) {
                    withCredentials([usernamePassword(credentialsId: 'nexus',
                        usernameVariable: 'NEXUS_USERNAME',
                        passwordVariable: 'NEXUS_PASSWORD')]) {
                        sh "mvn deploy -s $MAVEN_SETTINGS"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build succeeded — artifact deployed to Nexus!"
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
        failure {
            echo "❌ Build failed. Check console logs for details."
        }
    }
}
