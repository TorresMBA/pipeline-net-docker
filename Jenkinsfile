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

        stage('Build') {
            steps {
                sh 'dotnet build'
            }
        }

        stage('Test') {
            steps {
                sh 'dotnet test'
            }
        }
    }
}
