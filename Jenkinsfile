pipeline {
    agent any

    tools {
        nodejs "node18"
    }

    stages {

        stage('Install') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm run test -- --watch=false --browsers=ChromeHeadless'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build --prod'
            }
        }
    }
}
