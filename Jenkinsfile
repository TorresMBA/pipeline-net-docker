pipeline {
    agent any

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
    }
}
