pipeline {
    
    agent any
    
    environment {
        scannerHome = tool name: 'SonarQubeScanner'
        username = 'himanshubungla'
        dockerRegistry = 'himanshusb12/node-app'
        appPort = 7100
        dockerPort = 7100
    }
    
    tools {
        nodejs "node"
    }
    
    options {
        // prepend time stamps in log
        timestamps()
        
        // timeout
        timeout(time: 1, unit:'HOURS')
        
        // Skip checking out code from source control by default 
        skipDefaultCheckout()
        
        buildDiscarder(logRotator(
                numToKeepStr: '5',
                daysToKeepStr: '10'
            )
        )
        
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install dependencies') {
            steps {
                   bat 'npm install .'
                }
        }
        
        stage('Test cases') {
            steps {
                   bat 'npm test'
                
                }
        }
        
        stage('SonarQube Analysis') {
            steps {
                    withSonarQubeEnv('Test_Sonar') {
                    bat "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=node-app -Dsonar.projectName=node-app -Dsonar.language=js -Dsonar.sourceEncoding=UTF-8 -Dsonar.exclusions=tests/** -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info"
                } 
            }
        }
        
        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage("Create Docker image") {
            steps {
                bat "docker build . -t i-${username}-master"
            }
        }
        
        stage("Push to DockerHub") {
            steps {
                bat "docker tag i-${username}-master ${dockerRegistry}:${BUILD_NUMBER}"
                
                withDockerRegistry([credentialsId: 'DockerHub', url: '']) {
                    bat "docker push ${dockerRegistry}:${BUILD_NUMBER}"
                }
            }
        }
        
        stage("Docker deployment") {
            steps {
                echo 'Stopping the already running containers, if found'
                bat "for /F \"tokens=*\" %%n IN ('docker ps -a -q -f \"name=c-himanshubungla-master\"') DO docker rm -f %%n"

                echo 'Starting the api container'
                bat "docker run --name c-${username}-master -p ${appPort}:${dockerPort} -d ${dockerRegistry}:${BUILD_NUMBER}"
            }
        }
    }
}