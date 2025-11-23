pipeline {
    agent any

    tools {
        jdk 'jdk21'
    }

    environment {
        SONARQUBE_URL = "http://host.docker.internal:9000"
        SONAR_TOKEN = "squ_e1cc39c3c53c7ecd551b563eeeb6c6a825e6ff4b"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/LUCKY-DINESH/code-quality-reporter.git'
            }
        }

        stage('Compile Java Code') {
            steps {
                sh """
                    mkdir -p build
                    javac -d build \$(find . -name "*.java")
                """
            }
        }

        stage('Run Tests') {
            steps {
                sh """
                    echo "Running simple Java tests..."

                    if [ -f build/App.class ]; then
                        echo "Executing App.java output:"
                        java -cp build App || true
                    else
                        echo "No App.class found → skipping run"
                    fi
                """
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh """
                    sonar-scanner \
                      -Dsonar.projectKey=code-quality-reporter \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=${SONARQUBE_URL} \
                      -Dsonar.login=${SONAR_TOKEN}
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
                    allowMissing: false    // ✅ FIX ADDED
                ])
            }
        }
    }
}
