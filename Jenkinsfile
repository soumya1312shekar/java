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
                withSonarQubeEnv('SONAR') {
                    sh "mvn clean package sonar:sonar \
                        -Dsonar.projectKey=soumya1312shekar_java \
                        -Dsonar.organization=soumya1312shekar-1"
                }
            }
        }

        stage('Docker Push to ECR') {
            steps {
                sh """
                aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin ://amazonaws.com
                
                docker pull nginx:1.25
                docker tag nginx:1.25 ://amazonaws.com/dev/spcimage:latest
                docker push ://amazonaws.com/dev/spcimage:latest
                """
            }
        }

        stage('Deploy to K8s') {
            steps {
                withCredentials([file(credentialsId: 'myeks', variable: 'KUBECONFIG')]) {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG
                        # Use -f . to apply all files in the current folder if deploy-k8s is missing
                        kubectl apply -f . 
                    '''
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
            sh "docker logout ://amazonaws.com || true"
            cleanWs()
        }
    }
}
