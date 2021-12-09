pipeline {
    agent any
    stages {
        stage ('Build Backend') {
            steps {
                bat 'mvn clean package -DskipTests=true'
            }
        }
        stage ('Unit Tests') {
            steps {
                bat 'mvn test'
            }
        }
        stage ('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonar'
            }
            steps {
                withSonarQubeEnv('sonar') {
                   bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=29c21db8b90ab6f9b9c84c977d7f02d02d11c638 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/mvn/**,**/src/test/**,**/model/**,**Application.java"
                }    
            }
        }
        stage ('Quality Gate') {
            steps {
                sleep(10)
                timeout(time:1,unit:'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }                   
            }
        }
        stage ('Deploy Backend') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'tomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage ('API Tests') {
            steps {
                dir('api-test') {
                    git 'https://github.com/cassia-cristina/tasks-api-test.git'
                    bat 'mvn test'
                }
            }
        }
        stage ('Deploy Frontend') {
            steps {
                dir('frontend') {
                    git 'https://github.com/cassia-cristina/tasks-frontend.git'
                    bat 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'tomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }                
            }
        }
        stage ('Functional Tests') {
            steps {
                dir('function-test') {
                    git 'https://github.com/cassia-cristina/tasks-functional-test.git'
                    bat 'mvn test'
                }
            }
        }
    }
}