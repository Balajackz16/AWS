
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        EC2_HOST = '172.31.20.54'
        EC2_USER = 'ubuntu'
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
                sshagent(credentials: ['ec2-deploy-key']) {
                    sh '''
                        set -eu

                        scp -o StrictHostKeyChecking=yes \
                            index.html \
                            "$EC2_USER@$EC2_HOST:/tmp/index.html"

                        ssh -o StrictHostKeyChecking=yes \
                            "$EC2_USER@$EC2_HOST" \
                            'sudo -n /usr/bin/install -o root -g root -m 644 /tmp/index.html /var/www/html/index.html &&
                             sudo -n /usr/sbin/nginx -t &&
                             sudo -n /usr/bin/systemctl reload nginx &&
                             rm -f /tmp/index.html'

                        echo "Deployment completed successfully"
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
            echo 'FAILED: Check Console Output for the error'
        }
    }
}
