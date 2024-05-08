pipeline {
    agent any
    tools{
        maven 'maven'
    }
  
    stages {
        stage('Build product microservice') {
            steps {
                sh "mvn -v"
                sh "mvn clean install"
            }
        }
        stage('Sonar analysis and dependency check') {
            steps {
                withSonarQubeEnv(installationName: 'sonarserver'){
                    sh 'mvn sonar:sonar'
                }
                  }
        }
     
        stage('Build product docker image') {
            steps {
                script{
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_cred', passwordVariable: 'pwd', usernameVariable: 'user')]) {
                        sh "docker build -t ${user}/ecomm-product ."
                    }
                }
            }
        }
       
    }
}