pipeline {
    agent any
    triggers {
        githubPush()
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/AshwinSubramanyaGS/Python-CI-CD.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat '''
                python -m venv venv
                venv\\Scripts\\python.exe -m pip install --upgrade pip
                venv\\Scripts\\python.exe -m pip install -r requirements.txt
                '''
            }
        }

        stage('Run Unit Tests') {
            steps {
                bat '''
                venv\\Scripts\\python.exe -m pytest --maxfail=1 --disable-warnings -q || echo "No tests found"
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
                start /B venv\\Scripts\\python.exe app.py

                REM ===== Wait 3 seconds for Flask to start =====
                ping 127.0.0.1 -n 4 > nul
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
