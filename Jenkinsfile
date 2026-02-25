pipeline {
    agent {lable 'spc'}
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
                 withsonarqubeEnv('SONAR') {
                    sh '''mvn package sonar:sonar \
                           -Dsonar.projectkey=soumya1312shekar_java \
                           -Dsonar.organization=soumya1312shekar-1 \
                           -Dsonar.host.url=https://sonarcloud.io/ \
                           -Dsonar.login=$SONAR_TOKEN'''

                 



 

               
                     
                 }

                }
            }
            }

        }
}