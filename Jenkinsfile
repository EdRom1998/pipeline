pipeline {
    agent none

    environment {
        NEXUS_REGISTRY  = '153.51.166.83:8123'
        FULL_IMAGE_NAME = "153.51.166.83:8123/war/WAR:1.0"
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
                                credentialsId: '4d74607a-5636-48d1-805b-22c2aaef1a18',
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
                sshagent(credentials: ['212605aa-1a6b-40eb-8784-b728b258075a']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no root@${PROD_HOST} '
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