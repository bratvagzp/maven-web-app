pipeline {
    agent any

    environment {
        // HARBOR_CREDS = credentials('harbor_creds')
        HARBOR_DOCKER_REPO = '192.168.100.17:80'
    }

    stages {
        stage('Build') {
            steps {
                script {
                    sh 'mvn clean install'
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                script {
                    nexusArtifactUploader(
                        artifacts: [
                            [artifactId: 'maven-web-app', classifier: '', file: 'target/maven-web-app.war', type: 'war']
                        ],
                        credentialsId: 'harbor_creds',
                        //groupId: 'sreegroup',
                        harborUrl: '192.168.100.17:80',
                        harborVersion: '',
                        protocol: 'http',
                        repository: 'mta',
                        //version: '1.0-SNAPSHOT'
                    )
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker Image'
                sh "docker build -t $HARBOR_DOCKER_REPO/mta/maven-web-app:${BUILD_NUMBER} ."
            }
        }

        stage('Docker Login') {
            steps {
                echo 'Harbor Docker Repository Login'
                script {
                    withCredentials([usernamePassword(credentialsId: 'Harbor_creds', usernameVariable: 'admin', passwordVariable: 'PASS')]) {
                        sh "echo $PASS | docker login -u $admin --password-stdin $HARBOR_DOCKER_REPO"
                    }
                }
            }
        }

        stage('Docker Push') {
            steps {
                echo 'Pushing Image to Docker Repository'
                sh "docker push $HARBOR_DOCKER_REPO/mta/maven-web-app:${BUILD_NUMBER}"
            }
        }

        stage('Docker Pull and Run') {
            steps {
                echo 'Pulling Image from Docker Repository'
                sh "docker pull $HARBOR_DOCKER_REPO/maven-web-app:${BUILD_NUMBER}"

                echo 'Running Docker Container'
                sh "docker run -d -p 80:8085 --name new-app $HARBOR_DOCKER_REPO/maven-web-app:${BUILD_NUMBER}"
            }
        }
    }
}
