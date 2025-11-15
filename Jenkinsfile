pipeline {
    agent any

    tools {
        jdk 'jdk21'           // Must exist in Jenkins Global Tool Configuration
        maven 'maven'         // Must exist in Jenkins Global Tool Configuration
    }

    environment {
        JAVA_HOME = "/usr/lib/jvm/java-21-openjdk-amd64"
        PATH = "${JAVA_HOME}/bin:${PATH}"
        SONAR_SCANNER = tool 'sonarqube'   // Name must match Jenkins Tool Config
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/LUCKY-DINESH/code-quality-reporter.git'
            }
        }

        stage('Verify Java') {
            steps {
                sh "java -version"
            }
        }

        stage('SonarQube Analysis') {
    environment {
        JAVA_HOME = "/usr/lib/jvm/java-21-openjdk-amd64"
        PATH = "${JAVA_HOME}/bin:${PATH}"
    }
    steps {
        withSonarQubeEnv('sonarqube') {
            sh """
                java -version
                sonar-scanner \
                  -Dsonar.projectKey=code-quality-reporter \
                  -Dsonar.projectName=code-quality-reporter \
                  -Dsonar.sources=. \
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
                sh 'echo "<p>Analysis Successful</p>" >> report/index.html'
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


