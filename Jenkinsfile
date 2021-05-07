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
        stage('Build image') {
            agent any
            steps {
                script {
                  sh 'docker build --no-cache -t $IMAGE_REPO/$IMAGE_NAME:$IMAGE_TAG .'
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
	    stage('Deploy using ansible') {
		    agent { docker { image 'dirane/docker-ansible:latest' } }
			steps {
			    script {
				    sh ''' 
					    cd ansible
						ansible-playbook -i clients.yml helloworld.yml
					'''
				}
			}
		}
		stage('test application') {
		    agent { docker { image 'dirane/docker-ansible:latest' } }
			steps {
			    script {
				    sh ''' 
					    cd ansible
						ansible-playbook -i clients.yml test.yml
					'''
				}
			}
		}
    }
}
