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
                container ('python') {
                    sh 'python --version'
            }
        }
    }
}
}
