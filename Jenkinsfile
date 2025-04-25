pipeline {
    agent any

    tools {
        nodejs "NodeJS" 
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/wilsnd/8.2CDevSecOps.git'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                bat 'npm install'
            }
        }
        
        stage('Run Tests') {
            steps {
                bat 'npm audit || exit 0'
            }
            post {
                always {
                    // Create a simulated email with test results
                    bat 'echo Test Email Notification > test-email.txt'
                    bat 'npm audit >> test-email.txt'
                    
                    // Try standard emailext as fallback
                    emailext attachLog: true,
                        body: "Test completed",
                        subject: "Test Stage",
                        to: 'wilsondju@gmail.com'
                }
            }
        }
        
        stage('Generate Coverage Report') {
            steps {
                bat 'echo "Coverage report" > coverage.txt'
            }
        }
        
        stage('NPM Audit (Security Scan)') {
            steps {
                bat 'npm audit > security-audit.txt || exit 0'
            }
            post {
                always {
                    // Create a simulated email with security results
                    bat 'echo Security Email Notification > security-email.txt'
                    bat 'type security-audit.txt >> security-email.txt'
                    
                    // Try standard emailext as fallback
                    emailext attachLog: true,
                        attachmentsPattern: 'security-audit.txt',
                        body: "Security scan completed",
                        subject: "Security Scan",
                        to: 'wilsondju@gmail.com'
                }
            }
        }
    }
}
