pipeline {
    agent any
    environment {
        IMAGE_NAME = "ic-webapp"
        APP_CONTAINER_PORT = "8080"
        DOCKERHUB_ID = "choco1992"
        DOCKERHUB_PASSWORD = credentials('dockerhub_password')
        SCANNER_HOME=tool 'sonar-scanner'
    }
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
        stage("quality gate"){ 
           steps {
                script {
                    timeout(time: 2, unit: "MINUTES"){
                        waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                    }
                }
            } 
        }
        stage('OWASP FS SCAN') {
            steps {
                script{
                    dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                script{
                    sh 'trivy fs --format table -o trivy-fs-report.html .'
                }
            }
        }
        stage('Build image') {
           steps {
              script {
                sh 'docker build --no-cache -f ./sources/app/${DOCKERFILE_NAME} -t ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG ./sources/app'
                
              }
           }
       }
       stage('Run container based on builded image') {
          steps {
            script {
              sh '''
                  echo "Cleaning existing container if exist"
                  docker ps -a | grep -i $IMAGE_NAME && docker rm -f ${IMAGE_NAME}
                  docker run --name ${IMAGE_NAME} -d -p $APP_EXPOSED_PORT:$APP_CONTAINER_PORT  ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG
                  sleep 5
              '''
             }
          }
       }
       stage('Test image') {
           steps {
              script {
                sh '''
                   curl -I http://${HOST_IP}:${APP_EXPOSED_PORT} | grep -i "200"
                '''
              }
           }
       }
       stage('Clean container') {
          steps {
             script {
               sh '''
                   docker stop $IMAGE_NAME
                   docker rm $IMAGE_NAME
               '''
             }
          }
        }

       stage ('Login and Push Image on docker hub') {
          steps {
             script {
               sh '''
                   echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_ID --password-stdin
                   docker push ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG
               '''
             }
          }
        }

    }
}
