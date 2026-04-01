pipeline {
    agent { label 'spc' }
    
    environment {
        // AWS ECR Configuration
        AWS_ACCOUNT_ID = '271071982991'
        AWS_REGION     = 'ap-south-1'
        ECR_REPO_NAME  = 'java-app' // Ensure this repo is created in ECR Console
        ECR_REGISTRY   = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        
        // SonarCloud Configuration
        SONAR_ORG      = 'soumya1312shekar-1'
        SONAR_PROJECT  = 'soumya1312shekar_java'
    }

    triggers {
        pollSCM('* * * * *')
    }
 
    stages {
        stage('Git Checkout') {   
            steps {
                git url: 'https://github.com', branch: 'main'
            }
        }

        stage('Build and Scan') {
            steps {
                // 'sonar_sonar' must be a 'Secret text' type credential in Jenkins
                withCredentials([string(credentialsId: 'sonar_sonar', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('SONAR') {
                        sh """
                        mvn clean package sonar:sonar \
                        -Dsonar.projectKey=${SONAR_PROJECT} \
                        -Dsonar.organization=${SONAR_ORG} \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Docker Hub Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: '656587bb-ceb1-4f1a-918c-02aa85dcfd46',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                    docker build -t ${DOCKER_USER}/java-app:${BUILD_NUMBER} .
                    docker push ${DOCKER_USER}/java-app:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('ECR Push') {
            steps {
                sh """
                # Authenticate to AWS ECR
                aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                
                # Tag and Push to ECR (Fixed pathing)
                docker tag ${DOCKER_USER}/java-app:${BUILD_NUMBER} ${ECR_REGISTRY}/${ECR_REPO_NAME}:latest
                docker push ${ECR_REGISTRY}/${ECR_REPO_NAME}:latest
                """
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            sh "docker logout ${ECR_REGISTRY} || true"
            sh "docker logout || true"
        }
    }
}
