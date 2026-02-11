pipeline {
    agent any

    stages {

        stage('Clone') {
            steps {
                git branch: 'main',
                url: 'https://github.com/spring-projects/spring-petclinic.git'
            }
        }

        stage('Build + Test + Coverage') {
            steps {
                bat 'mvn clean verify'
            }
        }

        stage('SonarQube Analysis (properties inside pipeline)') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    bat '''
                    sonar-scanner ^
                      -Dsonar.projectKey=spring-petclinic ^
                      -Dsonar.projectName=Spring-PetClinic ^
                      -Dsonar.projectVersion=1.0 ^
                      -Dsonar.sources=src/main/java ^
                      -Dsonar.tests=src/test/java ^
                      -Dsonar.java.binaries=target/classes ^
                      -Dsonar.sourceEncoding=UTF-8 ^
                      -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package') {
            steps {
                bat 'mvn package -DskipTests'
            }
        }

        stage('Deploy') {
            steps {
                bat '''
                echo Killing old app on port 8085...
                for /F "tokens=5" %%a in ('netstat -ano ^| findstr :8085') do taskkill /F /PID %%a >nul 2>&1

                echo Starting application...
                powershell -Command "Start-Process java -ArgumentList '-jar target\\spring-petclinic-4.0.0-SNAPSHOT.jar --server.port=8085' -WindowStyle Hidden"

                ping 127.0.0.1 -n 16 >nul
                '''
            }
        }
    }

    post {
        success {
            emailext(
                subject: "BUILD SUCCESS",
                body: "Spring PetClinic pipeline executed successfully",
                to: "yourmail@gmail.com"
            )
        }

        failure {
            emailext(
                subject: "BUILD FAILED",
                body: "Spring PetClinic pipeline failed. Please check Jenkins logs.",
                to: "yourmail@gmail.com"
            )
        }
    }
}

