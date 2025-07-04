pipeline {
    environment{
        SCANNER_HOME=tool 'sonar-scanner'
    }
    agent any
    stages { 
       stage('Clone code from GitHub'){
           steps{
              git url: 'https://github.com/tafita00/ic-webapp.git', branch: 'main'   
           } 
       }
        stage("Sonarqube Analysis "){
            steps{
                script{
                    withSonarQubeEnv('sonar-server') {
                        sh '''
                            $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=wanderlust -Dsonar.projectKey=wanderlust
                        '''
                    }  
                }
            }
        }
        
    }  
}
