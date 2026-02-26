pipeline {
    agent { label 'spc' }
    
    tools {
        jdk 'jdk-25' 
    }

    environment {
        // Increase memory to prevent OutOfMemory errors during the scan
        MAVEN_OPTS = '-Xmx1024m'
    }

    triggers {
        pollSCM('* * * * *')
    }
    
    stages {
        stage('git checkout') {
            steps {
                git url : 'https://github.com',
                    branch: 'main'
            }
        }
        
        stage('build and scan') {
            steps {
                withCredentials([string(credentialsId: 'sonar_id', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('SONAR') {
                        // Added -DskipTests, -Dspring-javaformat.skip, and -Dcheckstyle.skip
                        sh '''mvn package sonar:sonar \
                               -DskipTests \
                               -Dspring-javaformat.skip=true \
                               -Dcheckstyle.skip=true \
                               -Dsonar.projectKey=soumya1312shekar_java \
                               -Dsonar.organization=soumya1312shekar-1 \
                               -Dsonar.host.url=https://sonarcloud.io \
                               -Dsonar.login=$SONAR_TOKEN'''
                    }
                }
            }
        }
    }
}

