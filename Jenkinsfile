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
                bat 'npm audit fix --force || exit 0' // auto fix error
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
                // Download SonarScanner (using a version compatible with Java 11)
                bat '''
                    curl -L -o sonar-scanner.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.6.2.2472-windows.zip
                    powershell -command "Expand-Archive -Path sonar-scanner.zip -DestinationPath . -Force"
                '''
                
                // Create sonar-project.properties file if it doesn't exist
                bat '''
                    echo # sonar-project.properties > sonar-project.properties
                    echo sonar.projectKey=wilsnd_8.2CDevSecOps >> sonar-project.properties
                    echo sonar.organization=wilsnd >> sonar-project.properties
                    echo sonar.host.url=https://sonarcloud.io >> sonar-project.properties
                    echo sonar.login=%%SONAR_TOKEN%% >> sonar-project.properties
                    echo sonar.sources=. >> sonar-project.properties
                    echo sonar.exclusions=node_modules/**,test/** >> sonar-project.properties
                    echo sonar.javascript.lcov.reportPaths=coverage/lcov.info >> sonar-project.properties
                    echo sonar.projectName=NodeJS Goof Vulnerable App >> sonar-project.properties
                    echo sonar.sourceEncoding=UTF-8 >> sonar-project.properties
                '''
                
                // Use withCredentials to properly inject token
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    bat '''
                        sonar-scanner-4.6.2.2472-windows\\bin\\sonar-scanner.bat -Dsonar.login=%SONAR_TOKEN%
                    '''
                }
            }
        }
    }
}
