pipeline {
    agent any

    environment {
		APP_NAME = 'mercury'
		IMAGE_TAG = "${BUILD_NUMBER}"
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
		        sh 'docker rm -f mercury-api || true'
		        sh 'docker run -d --name mercury-api -p 8082:8080 ${APP_NAME}:${IMAGE_TAG}'
			sh 'docker ps --filter name=mercury-api'
		    }
		}	
		stage('Health Check') {
		    steps {
		        script {
		            try {
		                sh '''
		                    echo "Waiting for application..."
		                    sleep 3
		
		                    curl --fail http://localhost:8082/health
		
		                    echo "Health Check OK"
		                '''		
		            } catch (Exception e) {
		
		                echo "❌ Health Check FAILED"
		                echo "Starting automatic rollback..."
		
		                def previousBuild = currentBuild.previousSuccessfulBuild
		
		                if (previousBuild == null) {
		                    error "No previous successful build available for rollback"
		                }
		
		                def previousTag = previousBuild.number.toString()
		
		                echo "Rolling back to mercury:${previousTag}"
		
		                sh """
		                    docker rm -f mercury-api || true
		
		                    docker run -d \
		                        --name mercury-api \
		                        -p 8082:8080 \
		                        ${APP_NAME}:${previousTag}
		
		                    echo "Waiting for rollback application..."
		                    sleep 3
		
		                    curl --fail http://localhost:8082/health
		                """
		
		                echo "✅ Rollback successful: mercury:${previousTag}"
		
		                throw e
		            }
		        }
	    	}
		}
	}
}
