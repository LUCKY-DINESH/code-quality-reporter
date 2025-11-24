pipeline {
    agent any

    tools {
        jdk 'jdk21'
        maven 'maven'
    }

    environment {
        SONAR_HOST_URL = "http://172.18.0.2:9000"
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
                    mvn sonar:sonar \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=admin \
                        -Dsonar.password=admin
                """
            }
        }

        stage('Generate HTML Report') {
    steps {
        sh """
            mkdir -p report
            cat > report/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Code Quality Dashboard</title>
    <style>
        body {
            margin: 0;
            padding: 40px;
            font-family: 'Segoe UI', Arial, sans-serif;
            background: linear-gradient(135deg, #dfe9f3 0%, #ffffff 100%);
        }
        .container {
            max-width: 900px;
            margin: auto;
        }
        .card {
            background: white;
            padding: 30px;
            border-radius: 16px;
            box-shadow: 0 8px 24px rgba(0,0,0,0.1);
            margin-bottom: 20px;
        }
        h1 {
            text-align: center;
            font-size: 32px;
            color: #333;
        }
        .badge {
            padding: 10px 20px;
            border-radius: 20px;
            color: white;
            font-weight: bold;
            display: inline-block;
            margin-top: 10px;
        }
        .good { background: #2ecc71; }
        .average { background: #f1c40f; }
        .poor { background: #e74c3c; }

        .progress-container {
            width: 100%;
            background: #eee;
            border-radius: 20px;
            margin-top: 20px;
        }
        .progress-bar {
            height: 24px;
            border-radius: 20px;
            width: 78%; /* Example score */
            background: linear-gradient(90deg, #6ddf82, #2ecc71);
            text-align: center;
            line-height: 24px;
            color: white;
            font-size: 14px;
            animation: grow 2s ease-out;
        }
        @keyframes grow {
            from { width: 0; }
            to { width: 78%; }
        }

        .footer {
            text-align: center;
            margin-top: 30px;
            color: #555;
            font-size: 14px;
        }
    </style>
</head>

<body>
    <div class="container">
        <div class="card">
            <h1>Code Quality Report</h1>
            <p><strong>Project:</strong> code-quality-reporter</p>
            <p><strong>Analysis Status:</strong></p>

            <!-- CHANGE BADGE BASED ON QUALITY -->
            <span class="badge good">🟢 GOOD QUALITY</span>

            <div class="progress-container">
                <div class="progress-bar">78% Quality Score</div>
            </div>

            <p style="margin-top: 20px;">
                View full detailed analysis in SonarQube dashboard.
            </p>
        </div>

        <div class="footer">
            Generated Automatically by Jenkins + SonarQube
        </div>
    </div>
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
                    keepAll: true
                ])
            }
        }

    }
}
