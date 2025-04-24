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
                bat 'npm audit fix --force || exit 0' // Fix vulnerabilities
            }
        }
        stage('Run Tests') {
            steps {
                bat 'npm test || exit 0' // Allows pipeline to continue despite test failures
            }
        }
        stage('Generate Coverage Report') {
            steps {
                // Add coverage script to package.json if it doesn't exist
                bat '''
                    echo "Adding coverage script to package.json if needed"
                    powershell -Command "$json = Get-Content package.json -Raw | ConvertFrom-Json; if (-not ($json.scripts.PSObject.Properties.Name -contains 'coverage')) { $json.scripts | Add-Member -NotePropertyName 'coverage' -NotePropertyValue 'nyc npm test'; $json | ConvertTo-Json -Depth 10 | Set-Content package.json }"
                '''
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
                // Download and extract SonarScanner version 4.2.0.1873 (older version compatible with Java 11)
                bat '''
                    curl -L -o sonar-scanner.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.2.0.1873-windows.zip
                    powershell -command "Expand-Archive -Path sonar-scanner.zip -DestinationPath . -Force"
                '''
                
                // Use withCredentials to properly inject token and run the CORRECT binary
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    bat '''
                        sonar-scanner-4.2.0.1873-windows\\bin\\sonar-scanner.bat -Dsonar.login=%SONAR_TOKEN%
                    '''
                }
            }
        }
    }
}
