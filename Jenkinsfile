pipeline {
    agent any

    tools {
        nodejs 'nodejs23'
    }

    environment{
        SCANNER_HOME = tool 'sonar-scanner'
        PROJECT_NAME = 'CICD_3tier_app'
        ENVIRONMENT = '🚀 Production'
    }

    stages {
        stage('Clean-WS') {
            steps {
                cleanWs()
            }
        }
        
        
        stage('Git-Checkout') {
            steps {
                git branch: 'dev', url: 'https://github.com/DevSecOpsProject-Org-nrjydv1997/3-Tier-DevSecOps.git'
            }
        }
        
        stage('Client-compilation') {
            steps {
                dir('client') {
                    sh 'find . -name "*.js" -exec node --check {} +'
                }
            }
        }
        
        stage('Api-compilation') {
            steps {
                dir('api') {
                    sh 'find . -name "*.js" -exec node --check {} +'
                }
            }
        }
        
        
        stage('GitLeaks-Scan') {
            steps {
                sh 'gitleaks detect --source ./client --exit-code 1'
                sh 'gitleaks detect --source ./api --exit-code 1'
            }
        }
        
        
        stage('SonarQube-Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=NodeJS-Project \
                    -Dsonar.projectKey=NodeJS-Project '''
                }
            }
        }
        
        
        stage('QualityGate Check') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        
        
        stage('TrivyFS scan') {
            steps {
                sh 'trivy fs --format table -o fs-report.html .'
            }
        }
        
        stage('Build & Tag Backend Docker image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        dir('api') {
                            sh 'docker build -t nrjydv1997/threetierdevsecopsbackend:latest .'
                            sh 'trivy image --format table -o backend-image-report.html nrjydv1997/threetierdevsecopsbackend:latest'
                            sh 'docker push nrjydv1997/threetierdevsecopsbackend:latest'
                        }
                    }
                }
            }
        }
        
        stage('Build & Tag Front Docker image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        dir('client') {
                            sh 'docker build -t nrjydv1997/threetierdevsecopsfrontend:latest .'
                            sh 'trivy image --format table -o frontendimage-report.html nrjydv1997/threetierdevsecopsfrontend:latest'
                            sh 'docker push nrjydv1997/threetierdevsecopsfrontend:latest'
                        }
                    }
                }
            }
        }
        
        stage('Docker Deploy via compose') {
            steps {
                script {
                    echo "docker compose up -d 'running'"
                    //sh 'docker compose up -d'
                }
            }
        }
        
        stage('k8s-deploy') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: 'dev-my-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'dev', restrictKubeConfigAccess: false, serverUrl: 'https://564544282C4698B4B626EF98CF85ADAE.gr7.ap-south-1.eks.amazonaws.com') {
                       sh 'kubectl apply -f k8s/sc.yaml -n dev'
                       sh 'kubectl apply -f k8s/mysql.yaml -n dev'
                       sh 'kubectl apply -f k8s/backend.yaml -n dev'
                       sh 'kubectl apply -f k8s/frontend.yaml -n dev'
                       sleep 30
                    }
                }
            }
        }
        
        
        stage('-verify-k8s-deploy') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: 'dev-my-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'dev', restrictKubeConfigAccess: false, serverUrl: 'https://564544282C4698B4B626EF98CF85ADAE.gr7.ap-south-1.eks.amazonaws.com') {
                       sh 'kubectl get pods -n dev'
                       sh 'kubectl get svc -n dev'
                    }
                }
            }
        }

        post {
        success {
            withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_URL')]) {
                script {
                    def message = """{
                        "text": "*✅ ${PROJECT_NAME} Build Successful!*",
                        "attachments": [
                            {
                                "color": "#36a64f",
                                "fields": [
                                    { "title": "Job", "value": "${env.JOB_NAME}", "short": true },
                                    { "title": "Build", "value": "#${env.BUILD_NUMBER}", "short": true },
                                    { "title": "Environment", "value": "${ENVIRONMENT}", "short": true }
                                ],
                                "footer": "Jenkins CI",
                                "footer_icon": "https://www.jenkins.io/images/logos/jenkins/jenkins.png",
                                "ts": ${System.currentTimeMillis() / 1000},
                                "actions": [
                                    {
                                        "type": "button",
                                        "text": "View Build",
                                        "url": "${env.BUILD_URL}",
                                        "style": "primary"
                                    }
                                ]
                            }
                        ]
                    }"""
                    sh """curl -X POST -H 'Content-type: application/json' --data '${message}' $SLACK_URL"""
                }
            }
        }

        failure {
            withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_URL')]) {
                script {
                    def message = """{
                        "text": "<!here> *❌ ${PROJECT_NAME} Build Failed!*",
                        "attachments": [
                            {
                                "color": "#FF0000",
                                "fields": [
                                    { "title": "Job", "value": "${env.JOB_NAME}", "short": true },
                                    { "title": "Build", "value": "#${env.BUILD_NUMBER}", "short": true },
                                    { "title": "Environment", "value": "${ENVIRONMENT}", "short": true }
                                ],
                                "footer": "Jenkins CI",
                                "footer_icon": "https://www.jenkins.io/images/logos/jenkins/jenkins.png",
                                "ts": ${System.currentTimeMillis() / 1000},
                                "actions": [
                                    {
                                        "type": "button",
                                        "text": "View Build Logs",
                                        "url": "${env.BUILD_URL}",
                                        "style": "danger"
                                    }
                                ]
                            }
                        ]
                    }"""
                    sh """curl -X POST -H 'Content-type: application/json' --data '${message}' $SLACK_URL"""
                }
            }
        }

        always {
            echo "🎯 Post-build notification sent"
        }
        
    }
}

