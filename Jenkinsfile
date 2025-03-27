pipeline {
    agent any

    stages {

        stage('Build & Test') {
            steps {
                script {
                    def branchName = env.BRANCH_NAME
                    sh 'make build'  // Adjust build command
                    sh 'make test'   // Adjust test command
                    
                    // Report build/test status to GitHub
                    githubNotify state: 'PENDING', description: 'Running tests', context: 'CI/CD Pipeline'
                    
                    if (currentBuild.result == 'FAILURE') {
                        githubNotify state: 'FAILURE', description: 'Build failed', context: 'CI/CD Pipeline'
                        error("Build failed")
                    } else {
                        githubNotify state: 'SUCCESS', description: 'Tests passed', context: 'CI/CD Pipeline'
                    }
                }
            }
        }

        stage('Pull Request Validation') {
            when {
                branch 'PR-*'  // Detect PR builds
            }
            steps {
                script {
                    sh 'make build'
                    sh 'make test'
                }
            }
        }

        stage('Staging Deployment') {
            when {
                branch 'staging'
            }
            steps {
                script {
                    sh "kubectl apply -f k8s/staging-deployment.yaml --kubeconfig=$KUBE_CONFIG"
                    sh "kubectl rollout status deployment/staging-service --kubeconfig=$KUBE_CONFIG"
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
                        git config --global user.email "jenkins@yourdomain.com"
                        git config --global user.name "Jenkins CI"
                        git checkout main
                        git merge --no-ff staging
                        git push origin main
                        '''
                    }
                }
            }
        }

        stage('Main Branch Deployment') {
            when {
                branch 'main'
            }
            steps {
                script {
                    sh "kubectl apply -f k8s/production-deployment.yaml --kubeconfig=$KUBE_CONFIG"
                    sh "kubectl rollout status deployment/production-service --kubeconfig=$KUBE_CONFIG"
                }
            }
        }

        stage('Scheduled Production Deployment') {
            when {
                allOf {
                    branch 'main'
                    expression { new Date().format('E') == 'Tue' } // Deploy only on Tuesdays
                }
            }
            steps {
                script {
                    sh "kubectl apply -f k8s/production-deployment.yaml --kubeconfig=$KUBE_CONFIG"
                    sh "kubectl rollout status deployment/production-service --kubeconfig=$KUBE_CONFIG"
                }
            }
        }
    }

    post {
        failure {
            script {
                githubNotify state: 'FAILURE', description: 'Pipeline failed', context: 'CI/CD Pipeline'
            }
        }
        success {
            script {
                githubNotify state: 'SUCCESS', description: 'Pipeline successful', context: 'CI/CD Pipeline'
            }
        }
    }
}
