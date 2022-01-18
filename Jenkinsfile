pipeline{
    agent any
    stages{
        stage('Build Backend'){
            steps{
                bat 'mvn clean package -DskipTests=true'
            }
        }
        stage('Unit Tests'){
            steps{
                bat 'mvn test'
            }
        }
        stage('Sonar Analisys'){
            environment {
                scannerhome = tool 'SONAR_SCANNER'
            }
            steps{
                withSonarQubeEnv('SONAR_LOCAL'){
                    bat "${scannerhome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://192.168.1.4:9000 -Dsonar.login=a8786f64e5e229210fb2d5ee985aa5c69348b58f -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                }
            }
        }
        stage('Quality Gate'){
            steps{
                sleep(5)
                timeout(1) {
                    waitForQualityGate abortPipeline: true, credentialsId: 'sonar_token'
                }
            }
        }
        stage('Deploy Backend'){
            steps{
                deploy adapters: [tomcat8(credentialsId: 'tomcat_login', path: '', url: 'http://192.168.1.4:8001')], contextPath: 'tasks-backend', onFailure: false, war: 'target/tasks-backend.war'
            }
        }
        stage('API Tests'){
            steps{
                dir('api-test') {
                    git credentialsId: 'github_login', url: 'https://github.com/almeidaRodrigo/tasks-api-test.git'
                    bat 'mvn test'
                }  
            }
        }
        stage('Deploy Frontend'){
            steps{
                dir('tasks-frontend'){
                    git credentialsId: 'github_login', url: 'https://github.com/almeidaRodrigo/tasks-frontend.git'
                    bat 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'tomcat_login', path: '', url: 'http://192.168.1.4:8001')], contextPath: 'tasks', onFailure: false, war: 'target/tasks.war'
                }
            }
        }
        stage('Functional Tests'){
            steps{
                dir('functional-test') {
                    git credentialsId: 'github_login', url: 'https://github.com/almeidaRodrigo/tasks-functional-tests.git'
                    bat 'mvn test'
                }  
            }
        }
    }
}


