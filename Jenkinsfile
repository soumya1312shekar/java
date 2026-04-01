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

        stage('Docker Pull and Scan') {
            steps {
                sh """
                # Pull image from Docker Hub
                docker image pull nginx:1.25
                
                # Run Trivy Scan using a Docker container
                # The -v flag mounts the docker socket so Trivy can see local images
                docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                aquasec/trivy:latest image --severity HIGH,CRITICAL --exit-code 1 nginx:1.25
                """
            }
        }

        stage('Docker Push to ECR') {
            steps {
                sh """
                # Login to AWS ECR
                aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 271071982991.dkr.ecr.ap-south-1.amazonaws.com
                
                # Tag and Push to your ECR repo
                docker tag nginx:1.25 271071982991.dkr.ecr.ap-south-1.amazonaws.com/dev/spcimage:latest
                docker push 271071982991.dkr.ecr.ap-south-1.amazonaws.com/dev/spcimage:latest
                """
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/*.jar', allowEmptyArchive: true
            junit '**/surefire-reports/*.xml'
            // Ensure logout from the specific registry
            sh "docker logout 271071982991.dkr.ecr.ap-south-1.amazonaws.com || true"
        }
    }
}
