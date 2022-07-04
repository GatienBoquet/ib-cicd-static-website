pipeline {
     environment {
       ID_DOCKER = "gatien94"
       IMAGE_NAME = "static-website-ib"
       IMAGE_TAG = "v1"  
       DOCKERHUB_PASSWORD = credentials('dockerhubpassword')
     }
     agent none
     stages {
         stage('Build image') {
             agent any
             steps {
                script {
                  sh 'docker build -t ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG .'
                }
             }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
               script {
                 sh '''
                    docker run --name $IMAGE_NAME -d -p 80:80 -e PORT=80 ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG
                    sleep 5
                 '''
               }
            }
       }
       stage('Test image') {
           agent any
           steps {
              script {
                sh '''
                    curl http://jenkins | grep -i "dimension"
                '''
              }
           }
      }
      stage('Clean Container') {
          agent any
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
          agent any
          steps {
             script {
               sh '''
                   echo $DOCKERHUB_PASSWORD | docker login -u $ID_DOCKER --password-stdin
                   docker push ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG
               '''
             }
          }
      }     
     
     
     stage('Prepare ansible environment') {
            agent any
            environment {
                PRIVATE_KEY = credentials('private_keys_jenkins')
            }
            steps {
                sh '''
                     cp  $PRIVATE_KEY  id_rsa
                     chmod 600 id_rsa
                '''
            }
     }          
          
     stage('Push image in staging and deploy it') { 
           agent any
           steps {
               script {
                 sh '''
                     cd $WORKSPACE/ansible && ansible-playbook playbooks/deploy_app.yml  --private-key ../id_rsa -e env=staging                   
                 '''
               }
           }
     }
     stage('Push image in production and deploy it') {
          when {
              expression { GIT_BRANCH == 'origin/master' }
          }
          agent any
          steps {
               script {
                 sh '''
                     cd $WORKSPACE/ansible && ansible-playbook playbooks/deploy_app.yml  --private-key ../id_rsa -e env=prod
                 '''
               }
          }
     }
          
     stage('Remove temp files') {
            agent any
            steps {
                sh '''
                     rm -fr $WORKSPACE/ansible/id_rsa
                '''
            }
     }            
  }
      post {
    always {
      script {
        discordSend description: 'From Jenkins Pipeline Build', footer: 'Footer Text', image: 'https://upload.wikimedia.org/wikipedia/commons/9/90/Airbus_A320-271N_NEO_D-AVVA.jpg', link: env.BUILD_URL, result: currentBuild.currentResult, unstable: false, title: JOB_NAME, webhookURL: 'https://discordapp.com/api/webhooks/993255292219441152/xnapWOk4CixSf8R8vfEhmoBHkazLl5zuDmhJX4rkg9qR9zrOfx1Urbv720HosyqetZ9B'
      }
    }  
  }
}
