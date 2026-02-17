pipeline {
    agent any

        tools {

            nodejs "node10"
            //sonarScanner "SonarScanner"   // name you set in Global Tool Config

        }

        environment {
            IMAGE_NAME = "hazemhendi/angular-app"
            IMAGE_TAG = "${BUILD_NUMBER}"
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
                        //sh 'sonar-scanner -Dsonar.projectKey=angular-app -Dsonar.sources=src'
                        // Get the path to the scanner installed in Jenkins
                        script {
                            def scannerHome = tool name: 'SonarQube', type: 'hudson.plugins.sonar.SonarRunnerInstallation'

                            // Run scanner using full path
                            sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=angular-app -Dsonar.sources=src"
                        }
            
                    }
                }
            }
        }
        /*
        stage("Quality Gate") {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false, webhookSecretId: 'sonarWebHook'
                }
            }
        }
        */
        stage('Docker Build') {
            steps {
                script {
                    //def IMAGE_TAG = "${env.BUILD_NUMBER}"
                    echo 'Création Image angular: '
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker logout
                        """
                    }
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
