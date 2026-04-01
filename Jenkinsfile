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
                // Ensure 'sonar_sonar' ID exists in Jenkins Credentials
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
                    // Replace 'your-ecr-repo-name' with your actual ECR repository name
                    def ecrRepo = "://amazonaws.com"
                    
                    sh """
                    docker image pull nginx:1.25
                    aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 271071982991.dkr.ecr.ap-south-1.amazonaws.com
                    
                    # Fixed: Correct tagging syntax (no :// and requires a repo path)
                    docker tag nginx:1.25 ${ecrRepo}:latest
                    docker push ${ecrRepo}:latest
                    """
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            // junit '**/target/surefire-reports/*.xml' // Uncomment if you have tests
            sh "docker logout || true"
        }
    }
}
