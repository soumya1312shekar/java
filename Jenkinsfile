pipeline {
    agent { label 'spc' }
    
    // Add the tools block here
    tools {
        jdk 'jdk-25' // This name must match the name in Manage Jenkins -> Tools
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
                        // Added -X for debugging if it fails again
                        sh mvn package sonar:sonar -DskipTests \
                            -Dsonar.projectKey=soumya1312shekar_java \
                            -Dsonar.organization=soumya1312shekar-1 \
                            -Dsonar.host.url=https://sonarcloud.io/ \
                            -Dsonar.login=$SONAR_TOKEN

                    }
                }
            }
        }
    }
}
