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
		sh 'docker build -t docker-demo:latest .'
	    }
	}
	stage('Docker Deploy') {
	    steps {
	        sh 'docker rm -f mercury-api || true'
	        sh 'docker run -d --name mercury-api -p 8082:8080 docker-demo:${IMAGE_TAG}'
		sh 'docker ps --filter name=mercury-api'
	    }
	}	
    }
}
