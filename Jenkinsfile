pipeline {
    agent any

    tools {
        // Must match Jenkins → Global Tool Configuration names
        jdk 'jdk21'
        maven 'maven'
        sonarScanner 'sonarqube'
    }

    environment {
        JAVA_HOME = "/usr/lib/jvm/java-21-openjdk-amd64"
        PATH = "${JAVA_HOME}/bin:${PATH}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/LUCKY-DINESH/code-quality-reporter.git'
            }
        }

        stage('Verify Java Installation') {
            steps {
                sh "java -version"
                sh "echo Using JAVA_HOME = $JAVA_HOME"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """
                        sonar-scanner \
                        -Dsonar.projectKey=code-quality-reporter \
                        -Dsonar.projectName=code-quality-reporter \
                        -Dsonar.sources=. \
                        -Dsonar.java.binaries=target \
                        -Dsonar.host.url=http://host.docker.internal:9000
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
                    alwaysLinkToLastBuild: true
                ])
            }
        }
    }
}
