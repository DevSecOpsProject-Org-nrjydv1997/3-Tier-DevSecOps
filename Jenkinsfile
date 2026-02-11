pipeline {
    agent any

    tools {
        nodejs 'nodejs23'
    }

    environment{
        SCANNER_HOME = tool 'sonar-scanner'
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
                    sh 'docker compose up -d'
                }
            }
        }
        
        
    }
}

