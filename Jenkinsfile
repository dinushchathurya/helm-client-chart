pipeline {
    
    agent any
    
    stages {
        
        stage('Checkout code') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/dinushchathurya/helm-multi-env.git']])
            }
        }
        
        stage('Generate Random Semantic Version') {
            steps {
                script {
                    version = sh(script: 'echo v0.$((1 + RANDOM % 9)).$((1 + RANDOM % 9))-${BUILD_NUMBER}', returnStdout: true).trim()
                    env.version = version
                }
            }
        }
        
        stage('Package Helm Chart') {
            agent {
                docker {
                    image 'limarktest/helm-client:0b0f49e'
                }
            }
            steps {
                sh 'helm version'
                sh 'helm package --version $version ./tomcat-chart'
            }
        }
        
        stage('Update chart repo index and chart') {
            agent {
                docker {
                    image 'limarktest/helm-client:0b0f49e'
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'GitHubCredentials', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    sh '''
                        helm repo index --merge index.yaml .
                        git config --local user.email "actions@github.com"
                        git config --local user.name "GitHub Actions"
                        git add .
                        git commit -m "feat: Update chart index with version $version"
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/dinushchathurya/helm-client-chart.git HEAD:master -f
                    '''
                }
            }
        }
        
    }
    
    post {
        cleanup {
            cleanWs(cleanWhenFailure: false)
        }
    }

}
