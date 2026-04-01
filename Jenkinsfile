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

        stage('Docker Hub Login & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: '656587bb-ceb1-4f1a-918c-02aa85dcfd46',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                    docker build -t ${DOCKER_USER}/java-app:${env.BUILD_ID} .
                    docker push ${DOCKER_USER}/java-app:${env.BUILD_ID}
                    """
                }
            }
        }

        stage('ECR Push') {
            steps {
                sh """
                aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 271071982991.dkr.ecr.ap-south-1.amazonaws.com
                docker tag ${DOCKER_USER}/java-app:${env.BUILD_ID} ://amazonaws.com{env.BUILD_ID}
                docker push ://amazonaws.com{env.BUILD_ID}
                """
            }
        }

        // --- NEW KUBERNETES INTEGRATION STAGE ---
        stage('Deploy to K8s') {
            steps {
                // 'eks-creds' should be a Jenkins credential of type 'AWS Credentials' or 'Kubeconfig'
                withKubeConfig(caCertificate: '', clusterName: 'my-eks-cluster', contextName: '', credentialsId: 'eks-creds', serverUrl: '') {
                    sh """
                    # Update the deployment image to the one we just pushed
                    kubectl set image deployment/java-app-deployment java-container=://amazonaws.com{env.BUILD_ID}
                    
                    # Verify the deployment
                    kubectl rollout status deployment/java-app-deployment
                    """
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/*.jar', allowEmptyArchive: true
            junit '**/surefire-reports/*.xml'
            sh "docker logout || true"
        }
    }
}
