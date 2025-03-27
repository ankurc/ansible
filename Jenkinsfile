pipeline {
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: python
                    image: python:3.10-slim-bullseye
                    command:
                    - sleep
                    args:
                    - infinity
                    tty: true
            '''
            defaultContainer 'python'
        }
    }

    stages {
        stage('Unit Test') {
            steps {
                sh 'apt-get update && apt-get install -y make'
                sh 'make test'
            }
        }
        // Add other stages as needed
    }
}
