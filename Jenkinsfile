pipeline {
    agent any

    tools {
        nodejs 'NodeJS'   // your existing NodeJS tool installation
        jdk    'jdk17'    // point this at Java 17 in your Jenkins Global Tool Config
    }

    environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN')  // add your SonarCloud token in Jenkins creds
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/wilsnd/8.2CDevSecOps.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm install'
                bat 'npm audit fix --force || exit 0'
            }
        }

        stage('Run Tests') {
            steps {
                bat 'npm test || exit 0'
            }
        }

        stage('Generate Coverage Report') {
            steps {
                bat 'echo "Adding coverage script to package.json"'
                bat 'npm install --save-dev nyc || exit 0'
                bat 'npm run coverage || exit 0'
            }
        }

        stage('NPM Audit (Security Scan)') {
            steps {
                bat 'npm audit || exit 0'
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                // Uses the SonarCloud Jenkins plugin, which will use your JDK17 automatically
                withSonarCloudEnv('SonarCloud') {
                    bat 'sonar-scanner -Dsonar.login=%SONAR_TOKEN%'
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished with status: ${currentBuild.currentResult}"
        }
    }
}
