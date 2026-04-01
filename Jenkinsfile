pipeline {
    agent { label 'spc' }
    
    environment {
        ECR_REGISTRY = "://amazonaws.com"
        REPO_NAME = "dev/spcimage"
    }
    
    triggers {
        pollSCM('* * * * *')
    }
 
    stages {
        // NOTE: You can remove this stage entirely because Jenkins already 
        // does 'Declarative: Checkout SCM' automatically at the start.
        stage('Git Checkout') {   
            steps {
                git url: 'https://github.com/soumya1312shekar/java.git', branch: 'main'
            }
        }

        stage('Build and Scan') {
            steps {
                withCredentials([string(credentialsId: 'sonar_sonar', variable: 'SONAR_TOKEN')]) {
                    // Changed to single quotes (''') and used $SONAR_TOKEN for security
                    sh '''
                        mvn clean package sonar:sonar \
                        -Dsonar.projectKey=soumya1312shekar_java \
                        -Dsonar.organization=soumya1312shekar-1 \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Docker Hub Login & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: '656587bb-ceb1-4f1a-918c-02aa85dcfd46', 
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker build -t "$DOCKER_USER/java-app:$BUILD_NUMBER" .
                        docker push "$DOCKER_USER/java-app:$BUILD_NUMBER"
                    '''
                }
            }
        }

        stage('ECR Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: '656587bb-ceb1-4f1a-918c-02aa85dcfd46', 
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                        docker tag ${DOCKER_USER}/java-app:${BUILD_NUMBER} ${ECR_REGISTRY}/${REPO_NAME}:latest
                        docker push ${ECR_REGISTRY}/${REPO_NAME}:latest
                    """
                }
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
