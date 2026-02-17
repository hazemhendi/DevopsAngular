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
    }
}
