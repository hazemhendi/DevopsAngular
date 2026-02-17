pipeline {
    agent any

        tools {
            nodejs "node10"
        }
    stages {

        stage('Install') {
            steps {
                dir('angular-app-kubernetes') {
                    sh 'node -v'
                    sh 'npm -v'
                    sh 'npm install'
                }
            }
        }

        stage('Run Tests') {
            steps {
                dir('angular-app-kubernetes') {
                    sh 'npm run test -- --watch=false --browsers=ChromeHeadless'
                }
            }
        }

        stage('Build') {
            steps {
                dir('angular-app-kubernetes') {
                    sh 'npm run build --prod'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('angular-app-kubernetes') {
                    withSonarQubeEnv('SonarQube') {  // Name from Jenkins config
                        sh 'sonar-scanner -Dsonar.projectKey=angular-app -Dsonar.sources=src'
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }


    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs!"
        }
    }
}
