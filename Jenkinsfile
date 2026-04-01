pipeline {
    agent { label 'spc' }
    
    environment {
        ECR_REGISTRY = "271071982991.dkr.ecr.ap-south-1.amazonaws.com"
        REPO_NAME    = "dev/spcimage"
        REGION       = "ap-south-1"
    }
    
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
                    sh """
                        mvn clean package sonar:sonar \
                        -Dsonar.projectKey=soumya1312shekar_java \
                        -Dsonar.organization=soumya1312shekar-1 \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }

        stage('Docker Pull & Push to ECR') {
    steps {
        sh """
            # Pull the official image
            docker pull nginx:1.25
            
            # Authenticate with AWS ECR
            aws ecr get-login-password --region ap-south-1 | \
            docker login --username AWS --password-stdin 271071982991.dkr.ecr.ap-south-1.amazonaws.com
            
            # Tag the image (Fixed typo: nginix -> nginx)
            docker tag nginx:1.25 271071982991.dkr.ecr.ap-south-1.amazonaws.com/dev/spcimage:latest
            
            # Push to your private repo
            docker push 271071982991.dkr.ecr.ap-south-1.amazonaws.com/dev/spcimage:latest
        """
    }
}

    
    post {
        always {
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            sh "docker logout ${ECR_REGISTRY} || true"
        }
    }
}
