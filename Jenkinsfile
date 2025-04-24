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
                // Modify coverage script to use a simpler approach
                bat 'echo "Adding coverage script to package.json"'
                bat 'npm install --save-dev nyc || exit 0'
                bat 'npm run coverage || exit 0'
            }
        }
        stage('NPM Audit (Security Scan)') {
            steps {
                bat 'npm audit || exit 0' // This will show known CVEs in the output
            }
        }
        // SonarQube with Java 17
        stage('SonarCloud Analysis') {
            steps {
                // Download Java 17
                bat '''
                    echo "Downloading JDK 17..."
                    curl -L -o jdk17.zip https://download.java.net/java/GA/jdk17.0.2/dfd4a8d0985749f896bed50d7138ee7f/8/GPL/openjdk-17.0.2_windows-x64_bin.zip
                    powershell -command "Expand-Archive -Path jdk17.zip -DestinationPath . -Force"
                '''
                
                // Download SonarScanner compatible with Java 17
                bat '''
                    echo "Downloading SonarScanner..."
                    curl -L -o sonar-scanner.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.0.2856-windows.zip
                    powershell -command "Expand-Archive -Path sonar-scanner.zip -DestinationPath . -Force"
                '''
                
                // Run SonarScanner with Java 17
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    bat '''
                        echo "Running SonarScanner with JDK 17..."
                        set "JAVA_HOME=%CD%\\jdk-17.0.2"
                        set "PATH=%JAVA_HOME%\\bin;%PATH%"
                        java -version
                        sonar-scanner-4.8.0.2856-windows\\bin\\sonar-scanner.bat -Dsonar.login=%SONAR_TOKEN%
                    '''
                }
            }
        }
    }
}
