pipeline {
    agent {lable 'spc'}
    stages {
        stage('git checkout') {
            steps {
                git url : 'https://github.com/sweetyvenni2013-ship-it/spring-framework-petclinic.git',
                    branch: 'main'
            }
            }
            stage('build and scan') {
                steps {
                    sh 'mvn package sonar:sonar'
                }
            }

        }
}