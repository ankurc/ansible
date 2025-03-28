pipeline {
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: python-builder
                image: python:3.10-slim
                command:
                - cat
                tty: true
              - name: ansible
                image: ansible/ansible-runner
                command:
                - cat
                tty: true
            '''
        }
    }
    
    environment {
        PACKAGES_DISK_PATH = '/home/ankur.chaudhary@outsyders.local/dev/pkg'
        PROD_DEPLOY_DAY = 'Tuesday'
    }
    
    stages {
        // Build and Test stage for all branches
        stage('Build and Test') {
            steps {
                container('python-builder') {
                    sh 'make build'
                    sh 'make test'
                }
            }
            post {
                always {
                    junit 'test-results.xml'
                    script {
                        currentBuild.result = currentBuild.result ?: 'SUCCESS'
                        githubNotify(
                            credentialsId: 'github-credentials-id',
                            status: currentBuild.result,
                            description: "Build ${currentBuild.result}"
                        )
                    }
                }
            }
        }
        
        // PR-specific stages
        stage('Pull Request Check') {
            when { 
                changeRequest() 
            }
            steps {
                echo "Building PR ${env.CHANGE_ID}"
            }
        }
        
        // Staging deployment stages
        stage('Staging Deployment') {
            when {
                branch 'staging'
            }
            stages {
                stage('Package Release') {
                    steps {
                        container('python-builder') {
                            sh "mkdir -p ${PACKAGES_DISK_PATH}"
                            sh "cp dist/*.whl ${PACKAGES_DISK_PATH}/"
                            sh "cp dist/*.tar.gz ${PACKAGES_DISK_PATH}/"
                        }
                    }
                }
                
                stage('Deploy FastAPI Service') {
                    when {
                        expression { fileExists('main.py') } // Assuming FastAPI service has main.py
                    }
                    steps {
                        container('ansible') {
                            sh '''
                            ansible-playbook -i inventory/staging deploy.yml \
                                -e "package_path=${PACKAGES_DISK_PATH}"
                            '''
                        }
                    }
                }
                
                stage('Merge Staging into Main') {
                    when {
                        branch 'staging'
                    }
                    steps {
                        script {
                            withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                                sh '''
                                git config --global user.email "ankur.chaudhary@theoutsyders.io"
                                git config --global user.name "Ankur Chaudhary"
                                git checkout main
                                git merge --no-ff staging
                                git push origin main
                                '''
                            }
                        }
                    }
                }
            }
        }
        
        // Production deployment stages
        stage('Production Deployment') {
            when {
                allOf {
                    branch 'main'
                    expression { 
                        new Date().format('EEEE') == env.PROD_DEPLOY_DAY
                    }
                }
            }
            stages {
                stage('Deploy to Production') {
                    steps {
                        container('ansible') {
                            sh '''
                            ansible-playbook -i inventory/production deploy.yml \
                                -e "package_path=${PACKAGES_DISK_PATH}"
                            '''
                        }
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            script {
                if (env.BRANCH_NAME == 'staging' || env.BRANCH_NAME == 'main') {
                    githubNotify(
                        credentialsId: 'github-credentials-id',
                        status: 'SUCCESS',
                        description: "Deployment completed successfully"
                    )
                }
            }
        }
        failure {
            script {
                if (env.BRANCH_NAME == 'staging' || env.BRANCH_NAME == 'main') {
                    githubNotify(
                        credentialsId: 'github-credentials-id',
                        status: 'FAILURE',
                        description: "Deployment failed"
                    )
                }
            }
        }
    }
}
