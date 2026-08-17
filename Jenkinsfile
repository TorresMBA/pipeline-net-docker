pipeline {
    agent any

    environment {
		APP_NAME = 'mercury'
		IMAGE_TAG = "${BUILD_NUMBER}"
		PATH = "/var/lib/jenkins/.dotnet/tools:${env.PATH}"
    }

    stages {

        stage('Environment') {
            steps {
                sh 'whoami'
                sh 'dotnet --version'
                sh 'docker --version'
            }
        }

        stage('Restore') {
            steps {
                sh 'dotnet restore'
            }
        }

        stage('Build') {
            steps {
                sh 'dotnet build -c Release --no-restore'
            }
        }

        stage('Test') {
            steps {
                sh 'dotnet test -c Release --no-build'
            }
        }

		stage('SonarQube Analysis') {
		    steps {
		        withSonarQubeEnv('SonarQube') {
		            sh '''
		                sonar-scanner \
		                  -Dsonar.projectKey=pipeline-net-docker \
		                  -Dsonar.sources=.
		            '''
		        }
		    }
		}

		stage('Quality Gate') {
		    steps {
		        timeout(time: 5, unit: 'MINUTES') {
		            waitForQualityGate abortPipeline: true
		        }
		    }
		}
		
        stage('Publish') {
            steps {
                sh 'dotnet publish -c Release -o ./publish --no-build --no-restore'
            }
        }
		stage('Docker Build'){
		    steps {
				sh 'docker build -t ${APP_NAME}:${IMAGE_TAG} .'
		    }
		}
		stage('Docker Deploy') {
		    steps {
		        sh '''
		            echo "Deploying GREEN..."
		
		            docker rm -f mercury-green || true
		
		            docker run -d \
		                --name mercury-green \
		                -p 8083:8080 \
		                ${APP_NAME}:${IMAGE_TAG}
		        '''
		    }
		}

		stage('Health Check GREEN') {
		    steps {
		        script {
		            try {
		                sh '''
		                    sleep 3
		                    curl --fail http://localhost:8083/health
		                '''
		            } catch (Exception e) {
		
		                sh 'docker rm -f mercury-green || true'
		
		                error "GREEN deployment failed"
		            }
		        }
		    }
		}

		stage('Switch to GREEN') {
		    steps {
		        sh '''
		            docker rm -f mercury-api || true
		
		            docker run -d \
		                --name mercury-api \
		                -p 8082:8080 \
		                ${APP_NAME}:${IMAGE_TAG}
		
		            docker rm -f mercury-green || true
		        '''
		    }
		}

		stage('Health Check Production') {
		    steps {
		        script {
		            try {
		                sh '''
		                    sleep 3
		                    curl --fail http://localhost:8082/health
		                '''
		            } catch (Exception e) {
		
		                def previousBuild = currentBuild.previousSuccessfulBuild
		
		                if (previousBuild == null) {
		                    error "No previous successful build available"
		                }
		
		                def previousTag = previousBuild.number.toString()
		
		                echo "Rolling back to mercury:${previousTag}"
		
		                sh """
		                    docker rm -f mercury-api || true
		
		                    docker run -d \
		                        --name mercury-api \
		                        -p 8082:8080 \
		                        ${APP_NAME}:${previousTag}
		
		                    sleep 3
		
		                    curl --fail http://localhost:8082/health
		                """
		
		                throw e
		            }
		        }
		    }
		}
	}
}
