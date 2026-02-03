pipeline {
    agent any

    tools {
        jdk 'JAVA_HOME'
        maven 'maven'
    }

    environment {
        TOMCAT_HOME = "C:\\Softwares\\apache-tomcat-9.0.112"
    }

    stages {

        stage('1. Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/spring-projects/spring-petclinic.git'
            }
        }

        stage('2. Build + Test + Coverage') {
            steps {
                bat 'mvn clean verify'
            }
        }

        stage('3. SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    bat 'mvn sonar:sonar'
                }
            }
        }

        stage('4. Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('5. Package WAR') {
            steps {
                bat 'mvn clean package -DskipTests'
            }
        }

        stage('6. Deploy to Tomcat') {
            steps {
                bat '''
                echo Stopping Tomcat...
                call "%TOMCAT_HOME%\\bin\\shutdown.bat"
                timeout /t 5

                echo Deploying WAR...
                del /f /q "%TOMCAT_HOME%\\webapps\\petclinic.war"
                rmdir /s /q "%TOMCAT_HOME%\\webapps\\petclinic"
                copy target\\*.war "%TOMCAT_HOME%\\webapps\\petclinic.war"

                echo Starting Tomcat...
                call "%TOMCAT_HOME%\\bin\\startup.bat"
                '''
            }
        }
    }

    post {
        success {
            mail to: 'elchuriyaswanth2@gmail.com',
                 subject: "✅ Jenkins Build SUCCESS - PetClinic",
                 body: "Congratulations Yaswanth 🎉\n\nYour Jenkins pipeline completed successfully!\n\nApplication deployed to Tomcat.\n\n-- Jenkins"
        }

        failure {
            mail to: 'elchuriyaswanth2@gmail.com',
                 subject: "❌ Jenkins Build FAILED - PetClinic",
                 body: "Hi Yaswanth,\n\nYour Jenkins pipeline FAILED.\n\nPlease check console logs.\n\n-- Jenkins"
        }
    }
}
