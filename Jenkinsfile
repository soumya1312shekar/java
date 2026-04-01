pipeline {
    agent { label 'spc' }
    
    triggers {
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

        stage('deploy to k8s for dev') {
            steps {
                // Securely uses your EKS credentials
                withCredentials([credentialsId: 'myeks']) {
                    sh 'kubectl apply -f deploy-k8s/.'
                }
            }
        }
    } // End of stages

    post {
        always {
            archiveArtifacts artifacts: '**/*.jar', allowEmptyArchive: true
            junit allowEmptyResults: true, testResults: '**/surefire-reports/*.xml'
            sh "docker logout 271071982991.dkr.ecr.ap-south-1.amazonaws.com || true"
        }
    }
}
