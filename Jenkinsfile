pipeline {
    environment {
        IMAGE_NAME = "ic-webapp"
        APP_CONTAINER_PORT = "8080"
        DOCKERHUB_ID = "choco1992"
        DOCKERHUB_PASSWORD = credentials('dockerhub_password')
        SCANNER_HOME=tool 'sonar-scanner'
    }
    agent none
    stages { 
       stage('Clone code from GitHub'){
           agent any 
           steps{
              git url: 'https://github.com/tafita00/ic-webapp.git', branch: 'main'   
           } 
       }
    }     
}
