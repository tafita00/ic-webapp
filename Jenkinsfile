pipeline {
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
