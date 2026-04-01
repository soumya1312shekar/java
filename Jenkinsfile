pipeline {
    agent { label 'spc' }
    
    triggers {
        // Poll every minute (Consider Webhooks for better performance)
        pollSCM('* * * * *')
    }
 
    stages {
        stage('Git Checkout') {   
            steps {
                // Using the specific 'git' step is fine, 
                // but 'checkout scm' is often preferred for multibranch pipelines
                git url: 'https://github.com/soumya1312shekar/java.git', branch: 'main'
            }
        }

        stage('Build and Scan') {
            steps {
                // In Jenkins, withSonarQubeEnv usually handles the URL and Token internally 
                // if configured in Global Tool Configuration.
                withSonarQubeEnv('SONAR') {
                    sh "mvn clean package sonar:sonar \
                        -Dsonar.projectKey=soumya1312shekar_java \
                        -Dsonar.organization=soumya1312shekar-1"
                }
            }
        }

        stage('Docker Push to ECR') {
            steps {
                // Use script block or direct shell for AWS login
                sh """
                aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 271071982991.dkr.ecr.ap-south-1.amazonaws.com
                
                docker pull nginx:1.25
                docker tag nginx:1.25 271071982991.dkr.ecr.ap-south-1.amazonaws.com/dev/spcimage:latest
                docker push 271071982991.dkr.ecr.ap-south-1.amazonaws.com/dev/spcimage:latest
                """
            }
        }

        stage('Deploy to K8s') {
            steps {
                // Ensure the 'kubernetes-cli' plugin is installed for withKubeConfig
                withKubeConfig(credentialsId: 'myeks') {
                    sh 'kubectl apply -f deploy-k8s/'
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
        }
        cleanup {
            // Securely logout and clean up the workspace
            sh "docker logout 271071982991.dkr.ecr.ap-south-1.amazonaws.com || true"
            cleanWs()
        }
    }
}
