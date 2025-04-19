pipeline {
    agent any
        
    tools {
        nodejs "NodeJS"  // Import the tools
    }

    environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN')  // Use the credential ID 
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/wilsnd/8.2CDevSecOps.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                bat 'npm install' // bat window
            }
        }
        stage('Run Tests') {
            steps {
                bat 'npm test || exit 0' // Allows pipeline to continue despite test failures
            }
        }
        stage('Generate Coverage Report') {
            steps {
                // Ensure coverage report exists
                bat 'npm run coverage || exit 0'
            }
        }
        stage('NPM Audit (Security Scan)') {
            steps {
                bat 'npm audit || exit 0' // This will show known CVEs in the output
            }
        }

        // SonarQube
        stage('SonarCloud Analysis') {
            steps {
                // Download SonarScanner (using a much older version)
                bat '''
                    curl -L -o sonar-scanner.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.2.0.1873-windows.zip
                    powershell -command "Expand-Archive -Path sonar-scanner.zip -DestinationPath . -Force"
                '''
                
                // Use withCredentials to properly inject token
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    bat '''
                        sonar-scanner-4.2.0.1873-windows\\bin\\sonar-scanner.bat -Dsonar.login=%SONAR_TOKEN%
                    '''
                }
            }
        }
    }
}
