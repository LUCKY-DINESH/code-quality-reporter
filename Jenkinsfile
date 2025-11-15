pipeline {
    agent any

    tools {
        maven 'maven'   // Jenkins → Global Tool Configuration → Maven
    }

    environment {
        SONAR_HOST_URL = 'http://host.docker.internal:9000'
        SONAR_AUTH_TOKEN = credentials('sonarqube') // Jenkins credential ID
    }

    stages {

        stage('Checkout Code') {
            steps {
                git 'https://github.com/harish981/sample.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh '''
                mvn verify sonar:sonar \
                    -Dsonar.projectKey=sample_project \
                    -Dsonar.host.url=$SONAR_HOST_URL \
                    -Dsonar.login=$SONAR_AUTH_TOKEN
                '''
            }
        }

        stage('Generate Quality Report') {
            steps {
                script {
                    def qg = waitForQualityGate()

                    if (qg.status == 'OK') {
                        writeFile file: 'code_quality.html', 
                        text: "<h1 style='color:green'>🟢 GOOD</h1>"
                    } 
                    else if (qg.status == 'WARN') {
                        writeFile file: 'code_quality.html', 
                        text: "<h1 style='color:orange'>🟡 AVERAGE</h1>"
                    } 
                    else {
                        writeFile file: 'code_quality.html', 
                        text: "<h1 style='color:red'>🔴 POOR</h1>"
                    }
                }
            }
        }
    }

    post {
        always {
            publishHTML(target: [
                reportDir: '.',
                reportFiles: 'code_quality.html',
                reportName: 'Code Quality Report'
            ])
        }
    }
}
