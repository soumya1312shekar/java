pipeline {
    agent { label 'spc' }

    tools {
        jdk 'jdk-17'
        maven 'Maven3'
    }

    environment {
        MAVEN_OPTS = '-Xmx1024m'
    }

    stages {
        stage('Checkout') {
    steps {
        git branch: 'main',
            credentialsId: 'github_token',
            url: 'https://github.com/soumya1312shekar/java.git'
            }
        }

        stage('Build & Sonar Scan') {
            steps {
                withCredentials([string(credentialsId: 'sonar_id', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('SONAR') {
                        sh '''
                        mvn clean verify sonar:sonar \
                        -DskipTests \
                        -Dsonar.projectKey=soumya1312shekar_java \
                        -Dsonar.organization=soumya1312shekar \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.login=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }
    }
}
   

