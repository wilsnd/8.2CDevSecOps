pipeline {
    agent any

    tools {
        nodejs "NodeJS" 
    }
    
    environment {
        EMAIL_RECIPIENTS = 'wilsondju@gmail.com'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[url: 'https://github.com/wilsnd/8.2CDevSecOps.git']],
                    extensions: [[$class: 'CleanBeforeCheckout']]
                ])
            }
        }
        
        stage('Install Dependencies') {
            steps {
                bat 'npm install'
            }
        }
        
        stage('Run Tests') {
            steps {
                // Capture test results
                script {
                    try {
                        bat 'npm test'
                    } catch (Exception e) {
                        currentBuild.result = 'UNSTABLE'
                        echo "Tests failed but continuing pipeline: ${e.message}"
                    }
                }
            }
            post {
                always {
                    // Fancyy Email
                    emailext attachLog: true,
                        attachmentsPattern: '**/test-results.xml,**/coverage/**',
                        body: '''<html>
                            <body>
                                <h2>Test Stage Report</h2>
                                <p><b>Build Status:</b> ${currentBuild.currentResult}</p>
                                <p><b>Build Number:</b> ${BUILD_NUMBER}</p>
                                <p><b>Build URL:</b> <a href="${BUILD_URL}">${JOB_NAME} [${BUILD_NUMBER}]</a></p>
                                <h3>Summary</h3>
                                <p>The test stage has completed with status: <b>${currentBuild.currentResult}</b></p>
                                <p>Please review the attached log file for detailed test results.</p>
                            </body>
                        </html>''',
                        mimeType: 'text/html',
                        subject: "[Jenkins] Test Stage: ${currentBuild.currentResult} - ${env.JOB_NAME} #${BUILD_NUMBER}",
                        to: "${env.EMAIL_RECIPIENTS}",
                        replyTo: "${env.EMAIL_RECIPIENTS}"
                }
            }
        }
        
        stage('Generate Coverage Report') {
            steps {
                bat 'npm run coverage || exit 0'
            }
        }
        
        stage('NPM Audit (Security Scan)') {
            steps {
                script {
                    try {
                        bat 'npm audit --json > security-audit.json || exit 0'
                        bat 'npm audit > security-audit.txt || exit 0'
                    } catch (Exception e) {
                        echo "Security scan failed but continuing pipeline: ${e.message}"
                    }
                }
            }
            post {
                always {
                    // RFancyy Email security report
                    emailext attachLog: true,
                        attachmentsPattern: '**/security-audit.*',
                        body: '''<html>
                            <body>
                                <h2>Security Scan Report</h2>
                                <p><b>Build Status:</b> ${currentBuild.currentResult}</p>
                                <p><b>Build Number:</b> ${BUILD_NUMBER}</p>
                                <p><b>Build URL:</b> <a href="${BUILD_URL}">${JOB_NAME} [${BUILD_NUMBER}]</a></p>
                                <h3>Summary</h3>
                                <p>The security scan has completed with status: <b>${currentBuild.currentResult}</b></p>
                                <p>Please review the attached security audit files for detailed information about vulnerabilities found in dependencies.</p>
                                <p style="color: red;"><b>Note:</b> These security findings should be reviewed by the security team.</p>
                            </body>
                        </html>''',
                        mimeType: 'text/html',
                        subject: "[Jenkins] Security Scan: ${currentBuild.currentResult} - ${env.JOB_NAME} #${BUILD_NUMBER}",
                        to: "${env.EMAIL_RECIPIENTS}",
                        replyTo: "${env.EMAIL_RECIPIENTS}"
                }
            }
        }
    }
    
    post {
        success {
            emailext attachLog: false,
                body: '''<html>
                    <body>
                        <h2>Build Successful!</h2>
                        <p>The pipeline completed successfully.</p>
                        <p><b>Build URL:</b> <a href="${BUILD_URL}">${JOB_NAME} [${BUILD_NUMBER}]</a></p>
                    </body>
                </html>''',
                mimeType: 'text/html',
                subject: "[Jenkins] Build Successful - ${env.JOB_NAME} #${BUILD_NUMBER}",
                to: "${env.EMAIL_RECIPIENTS}"
        }
        failure {
            emailext attachLog: true,
                body: '''<html>
                    <body>
                        <h2 style="color: red;">Build Failed!</h2>
                        <p>The pipeline has failed. Please investigate.</p>
                        <p><b>Build URL:</b> <a href="${BUILD_URL}">${JOB_NAME} [${BUILD_NUMBER}]</a></p>
                        <p>Check the attached log for details.</p>
                    </body>
                </html>''',
                mimeType: 'text/html',
                subject: "[Jenkins] Build Failed - ${env.JOB_NAME} #${BUILD_NUMBER}",
                to: "${env.EMAIL_RECIPIENTS}"
        }
    }
}
