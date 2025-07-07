pipeline {
    agent any
    environment {
        IMAGE_NAME = "ic-webapp"
        APP_CONTAINER_PORT = "8080"
        DOCKERHUB_ID = "tafitasoa0"
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
                            $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=ic-webapp -Dsonar.projectKey=ic-webapp
                        '''
                    }  
                }
            }
        }
        stage("quality gate"){ 
           steps {
                script {
                    timeout(time: 2, unit: "MINNUTES"){
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
       stage("TRIVY IMAGE"){
            steps{
                sh 'trivy image ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG > trivyimage.txt'
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
       stage('Deployment kubernetes'){
          steps {
             stage('Deployment ic-webapp'){
                script {
                    sh '''
                    kubectl apply -f ./sources/manifestes-k8s/ic-webapp
                    curl -I http://{HOST_IP}:30080 | grep -i "200"
                    kubectl apply -f ./sources/manifestes-k8s/postgres
                    kubectl apply -f ./sources/manifestes-k8s/odoo
                    curl -I http://{HOST_IP}:30069 | grep -i "200"
                    kubectl apply -f ./sources/manifestes-k8s/pgadmin
                    curl -I http://{HOST_IP}:30050 | grep -i "200"
                    '''
                }
             }  
           
          }
       }
        
    }
}


