pipeline {
    agent none

    environment {
        NEXUS_REGISTRY  = '153.51.166.83:8123'
        FULL_IMAGE_NAME = "153.51.166.83:8123/war/war:1.0"
        PROD_HOST       = '153.51.167.36'
        CONTAINER_NAME  = 'test_app'
    }

    stages {
        stage('Build & Push') {
            agent {
                dockerfile {
                    filename 'Dockerfile.agent'
                    args '-v /var/run/docker.sock:/var/run/docker.sock -u root'
                }
            }
            stages {
                stage('Checkout') {
                    steps {
                        git branch: 'master',
                            url: 'https://github.com/EdRom1998/war.git'
                    }
                }
                stage('Build Java') {
                    steps {
                        sh 'mvn clean package -DskipTests'
                    }
                }
                stage('Build Docker Image') {
                    steps {
                        sh "docker build -t ${FULL_IMAGE_NAME} ."
                    }
                }
                stage('Push to Nexus') {
                    steps {
                        withCredentials([usernamePassword(
                                credentialsId: '67c39064-04df-474f-9353-044da9b6d2f5',
                                usernameVariable: 'NEXUS_USER',
                                passwordVariable: 'NEXUS_PASS')]) {
                            sh "echo \$NEXUS_PASS | docker login ${NEXUS_REGISTRY} -u \$NEXUS_USER --password-stdin"
                        }
                        sh "docker push ${FULL_IMAGE_NAME}"
                        sh "docker rmi ${FULL_IMAGE_NAME} || true"
                    }
                }
            }
        }

        stage('Deploy to Prod (VM3)') {
            agent any
            steps {
                withCredentials([usernamePassword(
                                credentialsId: '67c39064-04df-474f-9353-044da9b6d2f5',
                                usernameVariable: 'NEXUS_USER',
                                passwordVariable: 'NEXUS_PASS')]) {
                    sh """
                        sshpass -p Chita1234! ssh -o StrictHostKeyChecking=no root@153.51.167.36 '
                            docker pull ${FULL_IMAGE_NAME}
                            docker stop ${CONTAINER_NAME} || true
                            docker rm   ${CONTAINER_NAME} || true
                            docker run -d --name ${CONTAINER_NAME} -p 8080:8080 ${FULL_IMAGE_NAME}
                        '
                    """
                }
            }
        }
    }
}