pipeline {
    agent any

    stages {
        stage('Unit Test') {
            agent {
                docker {
                    image 'python:3.13-slim-bullseye'
                    args '--user root' // Run as root to avoid permission issues
                    reuseNode true
                }
            }
            steps {
                sh 'apt-get update && apt-get install -y make'
                sh 'make test'
            }
        }
        // Add other stages here...
    }
}
