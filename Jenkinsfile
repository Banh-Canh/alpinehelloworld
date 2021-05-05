pipeline {
	environment {
		IMAGE_NAME = "alpinehelloworld"
		IMAGE_TAG = "latest"
		STAGING = "victor-staging"
		PRODUCTION = "victor-production"
		DOCKERHUB_LOGIN = "banhcanh"
	}
	agent none
	stages {
		stage('Build image') {
			agent any
			steps {
				script {
					sh 'docker build -t $DOCKERHUB_LOGIN/$IMAGE_NAME:$IMAGE_TAG .'
				}
			}
		}
		stage('Start Container') {
			agent any
			steps {
				script {
					sh 'docker run -it --name ${IMAGE_NAME} -p 5000:5000 -d -e PORT=5000 ${DOCKERHUB_LOGIN}/${IMAGE_NAME}:${TAG_NAME}'
				}
			}
		}
		stage('Test Container') {
			agent any
			steps {
				script {
					sh '''
					var=$(curl http://172.17.0.1:5000)
					if [ "$var" = "Hello world!" ]
					then
						exit 0
					else
						exit 1
					fi 
					'''
				}
			}
		}
		stage('Clean Container') {
			agent any
			steps {
				script {
					sh 'docker rm -fv ${IMAGE_NAME}
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
	}
}
