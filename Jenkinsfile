pipeline {
    agent any
    environment {
        DOCKER_CREDS = credentials('docker-credentials')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            agent {
                docker { image 'maven:3.6.3-openjdk-11-slim' }
            }
            steps {
                sh 'mvn clean install'
                archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            }
        }
        stage('SonarQube') {
             steps {
                 script {
                     def scannerHome = tool 'scanner-default'
                     withSonarQubeEnv('sonar-server') {
                         sh "${scannerHome}/bin/sonar-scanner \
                             -Dsonar.projectKey=labmaven02 \
                             -Dsonar.projectName=labmaven02 \
                             -Dsonar.sources=src/main/java \
                             -Dsonar.java.binaries=**/target/classes \
                             -Dsonar.tests=src/test/java"
                     }
                 }
            }
         }
        stage('Build Image') {
            steps {
                copyArtifacts filter: 'target/*.jar',
                              fingerprintArtifacts: true,
                              projectName: '${JOB_NAME}',
                              flatten: true,
                              selector: specific('${BUILD_NUMBER}'),
                              target: 'target/'
                sh 'docker --version'
                sh 'docker-compose --version'
                sh 'docker-compose build'
            }
        }
        stage('Publish Image') {
            steps {
                script {
                        sh 'docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}'
                        sh 'docker tag labmaven02 ${DOCKER_CREDS_USR}/labmaven02:$BUILD_NUMBER'
                        sh 'docker push ${DOCKER_CREDS_USR}/labmaven02:$BUILD_NUMBER'
                        sh 'docker logout'
                    }
            }
        }
        stage('Run Container') {
            steps {
                script {
                    sh 'docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}'
                    sh 'docker rm galaxyLabMaven -f'
                    sh 'docker run -d -p 8081:8080 --name galaxyLabMaven ${DOCKER_CREDS_USR}/labmaven02:$BUILD_NUMBER'
                    sh 'docker logout'
                }
            }
        }
        stage('Test Run Container') {
            steps {
                script {
                    sh 'docker ps'
                    // Agrega pruebas adicionales según sea necesario
                    // sh 'curl http://localhost:8080/your-endpoint'
                }
            }
        }
    }
}
