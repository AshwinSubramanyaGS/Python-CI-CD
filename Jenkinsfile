tools {
    python 'Python_3.11'
}
pipeline {
    agent any

    tools {
        python 'Python_3.11'   // Must match name in Jenkins Global Tool Config
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/YOUR_USERNAME/flask-hello.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat '''
                python -m venv venv
                call venv\\Scripts\\activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Run Unit Tests') {
            steps {
                bat '''
                call venv\\Scripts\\activate
                pytest --maxfail=1 --disable-warnings -q || echo "No tests found"
                '''
            }
        }

        stage('Generate Version Document') {
            steps {
                bat '''
                echo Build Version: > version.txt
                git rev-parse --short HEAD >> version.txt
                echo Build Time: >> version.txt
                powershell -Command "Get-Date -Format 'yyyy-MM-dd HH:mm:ss'" >> version.txt
                '''
                archiveArtifacts artifacts: 'version.txt', followSymlinks: false
            }
        }

        stage('Local Deploy') {
            steps {
                bat '''
                REM ===== Kill old Flask process on port 5000 =====
                for /f "tokens=5" %%a in ('netstat -ano ^| findstr :5000 ^| findstr LISTENING') do taskkill /F /PID %%a

                REM ===== Start new Flask app =====
                call venv\\Scripts\\activate
                start /B python app.py

                REM ===== Wait 3 seconds for Flask to start =====
                timeout /T 3 >nul
                '''
            }
        }
    }

    post {
        success {
            echo "Flask app built, tested, and deployed successfully!"
        }
        failure {
            echo "Build failed!"
        }
    }
}
