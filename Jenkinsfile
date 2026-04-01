pipeline {
    agent { label 'spc' }
    
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
                    // Removed withSonarQubeEnv wrapper because the plugin is missing on your server
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

        stage('Docker Hub Login & Push') {
            steps {
                // Ensure this ID matches your 'Username with password' credential in Jenkins
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
                script {
                    // Registry details from your image
                    def ecrRegistry = "271071982991.dkr.ecr.ap-south-1.amazonaws.com"
                    def repoName = "java-app" 
                    
                    sh """
                    # 1. Login to ECR
                    aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin ${ecrRegistry}
                    
                    # 2. Tag the image built in the previous stage for ECR
                    docker tag ${DOCKER_USER}/java-app:${BUILD_NUMBER} ${ecrRegistry}/${repoName}:latest
                    
                    # 3. Push to ECR
                    docker push ${ecrRegistry}/${dev/spcimage}:latest
                    """
                }
            }
        }
    }

    post {
        always {
            // Archive the JAR file produced by Maven
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            // Cleanup login sessions
            sh "docker logout || true"
        }
    }
}
