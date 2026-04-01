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
                        mvn clean package sonar:sonar \
                        -Dsonar.projectKey=soumya1312shekar_java \
                        -Dsonar.organization=soumya1312shekar-1 \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
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
                    // FIXED: Removed '://' and added the specific repository name
                    def ecrRegistry = "271071982991.dkr.ecr.ap-south-1.amazonaws.com"
                    def repoName = "java-app" // Make sure this matches your ECR repo name
                    
                    sh """
                    docker image pull nginx:1.25
                    aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin ${ecrRegistry}
                    
                    # Tagging with the full ECR path
                    docker tag nginx:1.25 ${ecrRegistry}/${repoName}:latest
                    docker push ${ecrRegistry}/${repoName}:latest
                    """
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            sh "docker logout || true"
        }
    }
}
