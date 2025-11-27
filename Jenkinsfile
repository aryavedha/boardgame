pipeline {
    agent any
    
    tools {
        jdk 'java17'
        maven 'maven3'
         }
         
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }     

    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/aryavedha/boardgame.git'
            }
        }
        stage('git leaks scan') {
            steps {
                sh 'gitleaks detect --source . -f json -r gitleaks-result.json --redact'
            }
        }
        stage('mvn compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('trivy scan') {
            steps {
                sh 'trivy fs -f table --severity HIGH,CRITICAL -o fs-scan-results.htm .'
            }
        }
        stage('unit test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('sonarqube analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh'''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=game \
                    -Dsonar.projectKey=game -Dsonar.java.binaries=target ''' 
                }
            }
        }
        stage('quality gate check') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('build package') {
            steps {
                sh 'mvn package'
            }
        }
        stage('deploy to nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'nexus-config', jdk: 'java17', maven: 'maven3', traceability: true) {
                   sh 'mvn deploy'
                }
            }
        }
        stage('build docker image & tag') {
            steps {
                sh 'docker build -t aryavedha/game:v1 .'
            }
        }
        stage('trivy image scan') {
            steps {
                sh 'trivy image -f table -o image-scan-results.htm aryavedha/game:v1'
            }
        }
        stage('docker push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh 'docker push aryavedha/game:v1'
                }
             }
          }
       }
    }
}

