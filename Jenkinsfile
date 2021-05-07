pipeline {
    environment {
       IMAGE_NAME = "helloworld"
       IMAGE_TAG = "latest"
       IMAGE_REPO = "guyduche"
	   STAGING = "aurelien-staging"
       PRODUCTION = "aurelien-production"
    }
    agent none
    stages {
		stage('Clone repo') {
            agent any
            steps {
                script {
                  sh '''
				    git clone https://github.com/guyduche/helloworld.git || echo "already cloned"
					cd helloworld || echo "already in helloworld"
				  '''
                }
            }
        }
        stage('Build image') {
            agent any
            steps {
                script {
                  sh 'docker build -t $IMAGE_REPO/$IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
                script {
                  sh '''
                    docker run --name $IMAGE_NAME -d -p 80:80 $IMAGE_REPO/$IMAGE_NAME:$IMAGE_TAG
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
                    curl http://172.17.0.1 | grep -q "Hello universe"
                  '''
                }
            }
        }
        stage('Clean Container') {
            agent any
            steps {
               script {
                 sh '''
                  docker rm -vf ${IMAGE_NAME}
                 '''
                }
            }
        }
        stage('Push image on dockerhub') {
            agent any
            environment {
                DOCKERHUB_LOGIN = credentials('dockerhub_id')
				DOCKERHUB_PASS = credentials('dockerhub_pass')
                
            }
            steps {
                script {
                   sh '''
		              docker login --username ${DOCKERHUB_LOGIN} --password ${DOCKERHUB_PASS}
                      docker push ${IMAGE_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
                   '''
                }
            }
        }
        stage('Push image in staging and deploy it') {
            when {
              expression { GIT_BRANCH == 'origin/master' }
            }
            agent any
            environment {
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps {
                script {
                    sh '''
                      heroku container:login
                      heroku create $STAGING || echo "project already exist"
                      heroku container:push -a $STAGING web
                      heroku container:release -a $STAGING web
                    '''
                }
            }
        }
		stage('Test staging deployment') {
		    agent any
		    steps {
			    script {
			       sh '''
				      curl https://${STAGING}.herokuapp.com | grep -q "Hello universe"
				   '''
			    }
		    }
		}
		stage('Push image in production and deploy it') {
		    when {
				expression { GIT_BRANCH == 'origin/master' }
				}
		    agent any
		    environment {
			    HEROKU_API_KEY = credentials('heroku_api_key')
		    }  
		    steps {
			    script {
				    sh '''
				      heroku container:login
				      heroku create $PRODUCTION || echo "project already exist"
				      heroku container:push -a $PRODUCTION web
				      heroku container:release -a $PRODUCTION web
				    '''
			    }
			}
		}
		stage('Test production deployment') {
		    agent any
		    steps {
			    script {
			       sh '''
				     curl https://${PRODUCTION}.herokuapp.com | grep -q "Hello universe"
				   '''
			    }
		    }
		}
    }
}
