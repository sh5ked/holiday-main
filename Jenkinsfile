pipeline {
    agent any

    stages {

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t holiday-events:1.0 .'
            }
        }

        stage('Push Image to GHCR') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'ghcr-credentials',
                    usernameVariable: 'GHCR_USER',
                    passwordVariable: 'GHCR_TOKEN'
                )]) {
                    sh '''
                        echo "$GHCR_TOKEN" | docker login ghcr.io -u "$GHCR_USER" --password-stdin
                        docker tag holiday-events:1.0 ghcr.io/sh5ked/holiday-events:1.0
                        docker push ghcr.io/sh5ked/holiday-events:1.0
                    '''
                }
            }
        }

        stage('Test SSH to Deployment Server') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'holiday-deploy-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                            -i "$SSH_KEY" \
                            "$SSH_USER@192.168.30.130" \
                            "hostname"
                    '''
                }
            }
        }

    }
}

