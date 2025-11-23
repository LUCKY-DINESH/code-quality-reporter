pipeline {
    agent any

    tools {
        maven 'maven'
        jdk 'jdk21'
    }

    environment {
        SONARQUBE_URL = "http://host.docker.internal:9000"
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
            environment {
                SONAR_TOKEN = credentials('sonarqube')  // Secret Text token
            }
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=code-quality-reporter \
                        -Dsonar.projectName=code-quality-reporter \
                        -Dsonar.host.url=${SONARQUBE_URL} \
                        -Dsonar.login=${SONAR_TOKEN}
                    """
                }
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
                            .pass { color: green; font-size: 20px; }
                            .header { font-size: 28px; font-weight: bold; }
                        </style>
                    </head>
                    <body>
                        <h1 class="header">SonarQube Code Quality Report</h1>
                        <p class="pass">Quality Gate: PASSED ✔</p>
                        <p>Project: code-quality-reporter</p>
                        <p>View full report in SonarQube Dashboard.</p>
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
                    allowMissing: false,          // REQUIRED BY YOUR PLUGIN
                    alwaysLinkToLastBuild: true,
                    keepAll: true
                ])
            }
        }
    }
}
