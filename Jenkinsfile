pipeline {
    agent { label 'spc' }
    
    triggers {
        pollSCM('* * * * *')
    }

    stages {
        stage('git checkout') {   
            steps {
                git url: 'https://github.com',
                    branch: 'main'
            }
        }

        stage('build and scan') {
            steps {
                withCredentials([string(credentialsId: 'sonar_sonar', variable: 'SONAR_TOKEN')]) {
                    // Use double quotes for the sh block to allow variable expansion
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

        stage('docker login & push') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'd3d6bc96-8f86-4e2e-acf6-ccb785b65c88',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        // Login to Docker
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                        
                        // Build the image (Assumes Dockerfile is in root)
                        sh "docker build -t ${DOCKER_USER}/java-app:${env.BUILD_ID} ."
                        
                        // Push the image
                        sh "docker push ${DOCKER_USER}/java-app:${env.BUILD_ID}"
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/*.jar'
            junit '**/surefire-reports/*.xml'
            
            // Clean up: Logout of Docker after build
            sh "docker logout"
        }
    }
}
