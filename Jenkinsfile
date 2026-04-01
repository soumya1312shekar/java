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
                // Re-using your specific credential ID
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
                // You must wrap this in withCredentials to access DOCKER_USER again
                withCredentials([usernamePassword(
                    credentialsId: '656587bb-ceb1-4f1a-918c-02aa85dcfd46', 
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    script {
                        def ecrRegistry = "271071982991.dkr.ecr.ap-south-1.amazonaws.com"
                        def repoName = "dev/spcimage" // Updated to match your image instructions
                        
                        sh """
                        # 1. Login to ECR (Username is ALWAYS 'AWS' for ECR)
                        aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin ${ecrRegistry}
                        
                        # 2. Tag the existing local image for ECR
                        docker tag ${DOCKER_USER}/java-app:${BUILD_NUMBER} ${ecrRegistry}/${repoName}:latest
                        
                        # 3. Push to ECR
                        docker push ${ecrRegistry}/${repoName}:latest
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            sh "docker logout ${ecrRegistry} || true"
            sh "docker logout || true"
        }
    }
}
