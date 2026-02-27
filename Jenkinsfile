pipeline {
    agent{label 'spc'}
    triggers{
        pollSCM('* * * * *')
    }
    stages {
        stage ('git checkout'){
            steps {
                git url:'https://github.com/soumya1312shekar/java.git',
                    branch:'main'
            }
        }
        stage ('build and scan'){
            steps{
                 withCredentials([string(credentialsId:'sonar_sonar', variable:'SONAR_TOKEN')]){
                    withSonarQubeEnv('SONAR'){
                        sh """ mvn package sonar:sonar \
                            -Dsonar.projectKey=soumya1312shekar_java \
                            -Dsonar.organization=soumya1312shekar-1 \
                            -Dsonar.host.url=https://sonarcloud.io/ \
                            -Dsonar.login=$SONAR_TOKEN  """

                    }
                
            }
                
            }

        }
    }

    post {
        always{
            archiveArtifacts artifacts: '**/*.jar'
            junit '**/surefire-reports/*.xml'

        }
    }
}

