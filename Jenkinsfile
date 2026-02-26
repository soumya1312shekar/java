pipeline {
    agent { label 'spc' }
    
    tools {
        jdk 'jdk-25' 
    }

    triggers {
        pollSCM('* * * * *')
    }
    
    stages {
        stage('git checkout') {
            steps {
                git url : 'https://github.com/soumya1312shekar/java.git',
                    branch: 'main'
            }
        }
        
        stage('build and scan') {
            steps {
                withCredentials([string(credentialsId: 'sonar_id', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('SONAR') {
                        sh '''mvn package sonar:sonar -DskipTests \
                               -Dsonar.projectKey=soumya1312shekar_java \
                               -Dsonar.organization=soumya1312shekar-1 \
                               -Dsonar.host.url=https://sonarcloud.io \
                               -Dsonar.login=$SONAR_TOKEN'''
                    }
                }
            }
        } // This correctly closes the stage
    } // This correctly closes the stages block
} // This correctly closes the pipeline block

