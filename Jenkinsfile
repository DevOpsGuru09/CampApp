pipeline {
    agent any
    tools {
        maven 'MAVEN'
        nodejs 'NodeJs'
    }

    environment {
        SCANNER_HOME = tool 'SONAR_SCAN'
        REMOTE_HOST = '192.168.1.9' // Replace with your Docker host IP
    }
    stages {   
        stage('Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/DevOpsGuru09/CampApp'
            }
        }    
        stage('Install Dependency Package') {
            steps {
                script {
                    sh 'npm install'
                }
            }
        } 
        stage('Unit Test') {
            steps {
                sh "npm test"
            }
        }
        stage('Trivy Scan') {
            steps {
                script {
                    sh 'docker run --rm -v $(pwd):/project aquasec/trivy fs --format table -o /project/fs-report.html /project'
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    sh '''
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectName=CampApp \
                        -Dsonar.projectKey=CampApp \
                        -Dsonar.sources=. \
                        -Dsonar.java.binaries=. \
                        -Dsonar.host.url=http://192.168.1.15:9002/ \
                        -Dsonar.login=squ_9b69eab749e454e3b6b15de42501a67bd99412ca

                    '''
                }
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                    dependencyCheck additionalArguments: '-s ./', odcInstallation: 'DP-Check'
                      dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Build & Push to Docker') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '48355594-1782-4bc9-b2d7-3470ed1322bb', toolName: 'Docker') {
                        sh 'docker build -t campapp .'
                        sh 'docker tag campapp scor8709/campapp:latest'
                    }
                }
            }
        }

        stage('Trivy Scan docker image') {
            steps {
                 script {
                    sh 'docker run --rm -v $(pwd):/project aquasec/trivy image --format table -o /project/image-report.html scor8709/campapp:latest'
                }
            }
        }

        stage('Push to Docker') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '48355594-1782-4bc9-b2d7-3470ed1322bb', toolName: 'Docker') {
                        sh 'docker push scor8709/campapp:latest'
                    }
                }
            }
        }
        stage('Deploy Container') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: '0ab2a435-eead-4006-a185-e907755fd565', 
                                                      usernameVariable: 'SSH_USERNAME', 
                                                      passwordVariable: 'SSH_PASSWORD')]) {
                        sh """
                            sshpass -p "$SSH_PASSWORD" ssh -o StrictHostKeyChecking=no $SSH_USERNAME@$REMOTE_HOST \\
                            'docker run -itd --name campapp -p 8072:3000 scor8709/campapp:latest'
                        """
                    }
                }
            }
        }
    }
}
