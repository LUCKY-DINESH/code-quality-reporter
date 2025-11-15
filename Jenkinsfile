pipeline {
    agent any

    tools {
        jdk 'JDK17'
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

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests=true'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonar') {
                    sh """
                        ${SONAR_SCANNER}/bin/sonar-scanner \
                        -Dsonar.projectKey=sample \
                        -Dsonar.projectName=sample \
                        -Dsonar.sources=./src \
                        -Dsonar.java.binaries=./target \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.login=admin \
                        -Dsonar.password=admin
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

        stage('Generate HTML Report') {
            steps {
                sh 'mkdir -p codequality'
                sh 'echo "<h1>Code Quality Summary</h1>" > codequality/index.html'
                sh 'echo "<p>SonarQube analysis completed successfully.</p>" >> codequality/index.html'
            }
        }

        stage('Publish HTML Report') {
            steps {
                publishHTML([
                    reportDir: 'codequality',
                    reportFiles: 'index.html',
                    reportName: 'Code Quality Report'
                ])
            }
        }
    }
}
