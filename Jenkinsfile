pipeline {
    agent any

    tools {
        jdk 'jdk21'
        maven 'maven'
    }

    environment {
        SONAR_HOST_URL = "http://172.18.0.2:9000"
        SONAR_TOKEN = "squ_e1cc39c3c53c7ecd551b563eeeb6c6a825e6ff4b"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/LUCKY-DINESH/code-quality-reporter.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn -B compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn -B test || true'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh """
                    mvn -B sonar:sonar \
                      -Dsonar.projectKey=code-quality-reporter \
                      -Dsonar.projectName=code-quality-reporter \
                      -Dsonar.sources=src \
                      -Dsonar.java.binaries=target/classes \
                      -Dsonar.host.url=${SONAR_HOST_URL} \
                      -Dsonar.login=${SONAR_TOKEN} \
                      -Dsonar.sourceEncoding=UTF-8
                """
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
                        <p>Open SonarQube to see full analysis.</p>
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

