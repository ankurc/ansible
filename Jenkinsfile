pipeline {
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: python
                    image: python:3.13-slim-bullseye
                    command:
                    - sleep
                    args:
                    - infinity
                    tty: true
                  tolerations:
                  - key: "node-role.kubernetes.io/control-plane"
                    operator: "Exists"
                    effect: "NoSchedule"
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
    }
}
