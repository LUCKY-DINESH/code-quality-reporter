pipeline {
    agent any

    tools {
        maven 'maven'
        jdk 'jdk21'
    }

    environment {
        SONARQUBE_URL = "http://host.docker.internal:9000"
        SONAR_TOKEN = "squ_e1cc39c3c53c7ecd551b563eeeb6c6a825e6ff4b"    // No Jenkins credentials used
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/LUCKY-DINESH/code-quality-reporter.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh "mvn clean compile test"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh """
                    mvn sonar:sonar \
                        -Dsonar.projectKey=code-quality-reporter \
                        -Dsonar.projectName=code-quality-reporter \
                        -Dsonar.host.url=${SONARQUBE_URL} \
                        -Dsonar.login=${SONAR_TOKEN}
                """
            }
        }

        stage('Wait for Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Generate HTML Report') {
            steps {
                sh """
                    mkdir -p report
                    cat > report/index.html << 'EOF'
                    <html>
                    <head>
                        <title>Code Quality Report</title>
                        <style>
                            body { font-family: Arial; padding: 20px; }
                            .header { font-size: 28px; font-weight: bold; }
                        </style>
                    </head>
                    <body>
                        <h1 class="header">SonarQube Code Quality Report</h1>
                        <p>Project: code-quality-reporter</p>
                        <p>Check SonarQube dashboard for full metrics.</p>
                    </body>
                    </html>
                    EOF
                """
            }
        }

        stage('Publish HTML Report') {
            steps {
                publishHTML([
                    reportDir: 'report',
                    reportFiles: 'index.html',
                    reportName: 'Code Quality Report',
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    allowMissing: false
                ])
            }
        }
    }
}
