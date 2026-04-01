pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "harinimuruges/spotify-devops:v2"
        EC2_IP = "13.238.29.37"
    }

    stages {

        stage('Build & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker build -t $DOCKER_IMAGE .
                    docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} '
                    docker stop spotify || true
                    docker rm spotify || true
                    docker pull ${DOCKER_IMAGE}
                    docker run -d -p 8081:80 --name spotify ${DOCKER_IMAGE}
                    '
                    """
                }
            }
        }
    }
}
