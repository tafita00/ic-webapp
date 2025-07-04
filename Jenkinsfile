pipeline {
    tools{
        nodejs  'NodeJs'
    }
    environment{
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
        stage("Sonarqube Analysis "){
            agent any 
            steps{
                script{
                    withSonarQubeEnv('sonar-server') {
                        sh '''
                            $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=uptime -Dsonar.projectKey=uptime
                        '''
                    }  
                }
            }
        }
        
    }  
}
