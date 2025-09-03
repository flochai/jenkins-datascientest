pipeline {
    agent any
    environment { 
      DOCKER_ID = "florianchaillou"
      DOCKER_IMAGE = "datascientestapi"
      DOCKER_TAG = "v.${BUILD_ID}.0" 
    }
    stages {
        stage('Building') {
          steps {
                sh '''
                python3 -m venv .venv
                . .venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
          }
        }
        stage('Test') {
          steps {
                sh '''
                . .venv/bin/activate
                python -m unittest
                '''
          }
        }
        stage('Deploy') {
          steps {
            script {
              sh '''
              docker rm -f jenkins
              docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
              docker run -d -p 8000:8000 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
              '''
            }
          }
        }
        stage('User Acceptance') {
          steps {
            input message: "Proceed to push to main", ok: "Yes"
          }
        }
        stage('Push & Merge') {
          parallel {
            stage('Push Image') {
              environment {
                DOCKERHUB_CREDENTIALS = credentials('DOCKER_PASS')
              }
              steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKER_ID --password-stdin'
                sh 'docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG'
              }
            }
            stage('Merge') {
              steps {
                echo 'Merging done'
              }
            }
          }
        }
    }
    post {
      always {
        sh 'docker logout'
      }
    }
}
