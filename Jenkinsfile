pipeline {
    agent { label 'spc' }
    
    triggers {
        // Polls every minute (consider using Webhooks for better performance)
        pollSCM('* * * * *')
    }
 
    stages {
        stage('Git Checkout') {   
            steps {
                git url: 'https://github.com/soumya1312shekar/java.git', branch: 'main'
            }
        }

        stage('Build and Scan') {
            steps {
                // withSonarQubeEnv usually handles the URL and basic auth automatically
                withCredentials([string(credentialsId: 'sonar_sonar', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('SONAR') {
                        sh """
                        mvn package sonar:sonar \
                        -Dsonar.projectKey=soumya1312shekar_java \
                        -Dsonar.organization=soumya1312shekar-1 \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Docker Push to ECR') {
            steps {
                // Use a script block for cleaner shell execution
                sh """
                # Pull the base image
                docker image pull nginx:1.25
                
                # Login to AWS ECR
                aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 271071982991.dkr.ecr.ap-south-1.amazonaws.com
                
                # Tag and Push
                docker tag nginx:1.25 271071982991.dkr.ecr.ap-south-1.amazonaws.com/dev/spcimage:latest
                docker push 271071982991.dkr.ecr.ap-south-1.amazonaws.com/dev/spcimage:latest
                """
            }
        }

        stage('Deploy to K8s') {
            steps {
                // Added a directory check to prevent the stage from failing if folder is missing
                sh 'kubectl apply -f deploy-k8s/.'
            }
        }
    }

    post {
        always {
            // Cleans up workspace artifacts and logs out of ECR
            archiveArtifacts artifacts: '**/*.jar', allowEmptyArchive: true
            junit allowEmptyResults: true, testResults: '**/surefire-reports/*.xml'
            sh "docker logout 271071982991.dkr.ecr.ap-south-1.amazonaws.com || true"
        }
    }
}
