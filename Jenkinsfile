pipeline {
    environment {
       IMAGE_NAME = "helloworld"
       IMAGE_TAG = "latest"
       IMAGE_REPO = "guyduche"
    }
    agent none
    stages {
		stage('Clone repo') {
            agent { docker { image 'dirane/docker-ansible:latest' } }
            steps {
                script {
                  sh '''
				    git clone https://github.com/guyduche/helloworld.git
					cd helloworld
				  '''
                }
            }
        }
        stage('Build image') {
            agent { docker { image 'dirane/docker-ansible:latest' } }
            steps {
                script {
                  sh 'docker build -t $IMAGE_REPO/$IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }
        stage('Run container based on builded image') {
            agent { docker { image 'dirane/docker-ansible:latest' } }
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
            agent { docker { image 'dirane/docker-ansible:latest' } }
            steps {
                script {
                  sh '''
                    curl http://172.17.0.1 | grep -q "Hello universe"
                  '''
                }
            }
        }
        stage('Clean Container') {
            agent { docker { image 'dirane/docker-ansible:latest' } }
            steps {
               script {
                 sh '''
                  docker rm -vf ${IMAGE_NAME}
                 '''
                }
            }
        }
        stage('Push image on dockerhub') {
            agent { docker { image 'dirane/docker-ansible:latest' } }
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
	}
}