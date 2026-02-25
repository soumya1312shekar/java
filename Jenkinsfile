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
                  withsonarqubeEnv('SONAR') {

                  }    
                    sh 'mvn package sonar:sonar'
                }
            }

        }
}