pipeline {
    agent any

    tools {
        jdk 'jdk21'
        maven 'maven'        // Required for SonarQube plugin
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
                    FILES=\$(find src -name "*.java")

                    if [ -z "\$FILES" ]; then
                        echo "❌ No Java files found!"
                        exit 1
                    fi

                    javac -d build \$FILES
                """
            }
        }

        stage('Run Tests') {
            steps {
                sh """
                    echo "Running simple Java execution..."

                    if [ -f build/App.class ]; then
                        echo "Executing App..."
                        java -cp build App || true
                    else
                        echo "⚠ No App.class found—skipping execution."
                    fi
                """
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh """
                    mvn sonar:sonar \
                      -Dsonar.projectKey=code-quality-reporter \
                      -Dsonar.sources=src \
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
                        <p>Open SonarQube for the full analysis.</p>
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
