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

         // This stage pauses the pipeline and waits for you to click "Proceed"

        /*
        // Generate a Google App Password for this to work 

        stage('Wait for Approval') {
            steps {
                script {
                    // Send email notification that approval is needed
                    emailext (
                        subject: "Jenkins Pipeline Needs Approval: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: """<p>Pipeline is waiting for your approval to merge to main and build Docker image.</p>
                                <p>Click <a href="${env.BUILD_URL}input">here</a> to approve or abort.</p>""",
                        to: 'hazeam22@gmail.com' // Replace with your email
                    )
                    
                    // Pause execution until user input
                    input message: 'Approve Merge to Main and Docker Build?', ok: 'Proceed'
                }
            }
        }
        */
/*
        stage('Merge to Main') {
            steps {
                script {
                    // Logic to merge dev to main
                    withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh """
                            git config user.email "MohamedHazem.HENDI@esprit.tn"
                            git config user.name "Jenkins CI"
                            
                            // to fech te main branche 
                            git fetch --all

                            //git checkout main
                            // 2. Create/Reset local 'main' to match 'origin/main'
                            // -B is safer than checkout because it creates the branch if it doesn't exist
                            git checkout -B main origin/main
                            git pull origin main

                            git merge dev

                            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/mohamedhazemhendi/DevopsAngular.git main
                        """
                    }
                }
            }
        }
        */
        stage('Merge to Main') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh '''
                            # Set config
                            git config user.email "MohamedHazem.HENDI@esprit.tn"
                            git config user.name "Jenkins CI"

                            # 1. Fetch all remote branches (essential so Jenkins knows 'main' exists)
                            #git fetch --all
                            
                            # Fetch ALL branches explicitly
                            git fetch origin +refs/heads/*:refs/remotes/origin/*

                            # 2. Create/Reset local 'main' to match 'origin/main'
                            # -B is safer than checkout because it creates the branch if it doesn't exist
                            git checkout -B main origin/main

                            # 3. Merge dev. 
                            # We use 'origin/dev' because local 'dev' might not be fully tracked
                            git merge origin/dev

                            # 4. Push
                            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/mohamedhazemhendi/DevopsAngular.git main
                        '''
                    }
                }
            }
        }


        stage('Docker Build') {
            steps {
                dir('angular-app-kubernetes') {
                    script {
                        //def IMAGE_TAG = "${env.BUILD_NUMBER}"
                        echo 'Création Image angular: '
                        sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                    }
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                dir('angular-app-kubernetes') {
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

    }


    post {
        success {
            mail to: 'hazeam22@gmail.com',
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """<p>Pipeline completed successfully!</p>
                        <p>Image pushed: ${IMAGE_NAME}:${IMAGE_TAG}</p>"""
                
            
        }
        failure {
            mail to: 'hazeam22@gmail.com',
                subject: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """<p>Pipeline failed. Check the logs at <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                        <p>Stage failed: ${env.STAGE_NAME}</p>"""
                
            
        }
    }
}
