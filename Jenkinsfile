pipeline {
    agent any

    tools {
        // Make sure in Jenkins → Global Tool Configuration
        // You configure a JDK and name it "JDK17"
        jdk 'JDK24'

        // Also configure Maven and name it "MAVEN3"
        maven 'MAVEN3'
    }

    environment {
        SONAR_SCANNER = tool 'SonarScanner'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/LUCKY-DINESH/code-quality-reporter.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonar') {
                    sh """
                        ${SONAR_SCANNER}/bin/sonar-scanner \
                        -Dsonar.projectKey=code-quality-reporter \
                        -Dsonar.projectName=code-quality-reporter \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://localhost:9000
                    """
                }
            }
        }

        stage('Wait for Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Create HTML Report') {
            steps {
                sh 'mkdir -p report'
                sh 'echo "<h1>Code Quality Report</h1>" > report/index.html'
                sh 'echo "<p>SonarQube Quality Gate Passed.</p>" >> report/index.html'
            }
        }

        stage('Publish HTML Report') {
            steps {
                publishHTML([
                    reportDir: 'report',
                    reportFiles: 'index.html',
                    reportName: 'Code Quality Report',
                    keepAll: true,
                    alwaysLinkToLastBuild: true,
                    allowMissing: false
                ])
            }
        }
    }
}

