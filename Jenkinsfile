
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        EC2_HOST = '172.31.20.54'
    }

    stages {
        stage('Checkout GitHub') {
            steps {
                checkout scm
                sh 'test -f index.html'
            }
        }

        stage('Deploy index.html') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ec2-deploy-key',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                        set -eu

                        scp -i "$SSH_KEY" \
                            -o IdentitiesOnly=yes \
                            -o StrictHostKeyChecking=yes \
                            index.html \
                            "$SSH_USER@$EC2_HOST:/tmp/index.html"

                        ssh -i "$SSH_KEY" \
                            -o IdentitiesOnly=yes \
                            -o StrictHostKeyChecking=yes \
                            "$SSH_USER@$EC2_HOST" \
                            'sudo -n /usr/bin/install -o root -g root -m 644 /tmp/index.html /var/www/html/index.html &&
                             sudo -n /usr/sbin/nginx -t &&
                             sudo -n /usr/bin/systemctl reload nginx &&
                             rm -f /tmp/index.html'

                        echo "Deployment successful"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: index.html deployed to EC2'
        }
        failure {
            echo 'FAILED: Check Console Output'
        }
    }
}
