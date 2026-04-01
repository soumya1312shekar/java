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

        stage('Build & Push to ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${REGION} | \
                    docker login --username AWS --password-stdin ${ECR_REGISTRY}

                    docker build -t ${REPO_NAME} .

                    docker tag ${REPO_NAME}:latest ${ECR_REGISTRY}/${REPO_NAME}:latest

                    docker push ${ECR_REGISTRY}/${REPO_NAME}:latest
                """
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            sh "docker logout ${ECR_REGISTRY} || true"
        }
    }
}
