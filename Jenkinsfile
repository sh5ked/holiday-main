pipeline {
    agent any

    triggers {
        pollSCM('H/1 * * * *')
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                        -t holiday-events:${BUILD_NUMBER} \
                        .
                '''
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
                        echo "$GHCR_TOKEN" | docker login ghcr.io \
                            -u "$GHCR_USER" \
                            --password-stdin

                        docker tag \
                            holiday-events:${BUILD_NUMBER} \
                            ghcr.io/sh5ked/holiday-events:${BUILD_NUMBER}

                        docker push \
                            ghcr.io/sh5ked/holiday-events:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Ansible Deployment') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'ghcr-credentials',
                        usernameVariable: 'GHCR_USER',
                        passwordVariable: 'GHCR_TOKEN'
                    ),
                    sshUserPrivateKey(
                        credentialsId: 'holiday-deploy-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                        ANSIBLE_HOST_KEY_CHECKING=False \
                        ansible-playbook \
                        -i inventory.ini \
                        --private-key "$SSH_KEY" \
                        -e "image_tag=${BUILD_NUMBER}" \
                        deploy.yml
                    '''
                }
            }
        }

    }
}

