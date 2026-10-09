pipeline {
    agent any

    options {
        timestamps()
    }

    environment {
        DEPLOY_HOST = '172.31.20.54'
        DEPLOY_USER = 'ubuntu'
        SSH_CREDENTIAL_ID = 'ec2-deploy-key'
    }

    stages {

        stage('Checkout GitHub Code') {
            steps {
                checkout scm

                sh '''
                    set -eu
                    echo "Files in the GitHub repository:"
                    ls -la
                    test -f index.html
                    echo "index.html found successfully"
                '''
            }
        }

        stage('Deploy index.html to EC2') {
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

                        echo "Copying index.html to deployment server..."

                        chmod 600 "$SSH_KEY"

                        scp \
                          -i "$SSH_KEY" \
                          -o IdentitiesOnly=yes \
                          -o StrictHostKeyChecking=yes \
                          index.html \
                          "$SSH_USER@$DEPLOY_HOST:/tmp/index.html"

                        echo "index.html copied successfully"
                    '''
                }
            }
        }

        stage('Configure Website and Reload Nginx') {
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

                        echo "Installing index.html into Nginx web root..."

                        ssh \
                          -i "$SSH_KEY" \
                          -o IdentitiesOnly=yes \
                          -o StrictHostKeyChecking=yes \
                          "$SSH_USER@$DEPLOY_HOST" \
                          'set -eu
                           sudo -n /usr/bin/install -o root -g root -m 644 /tmp/index.html /var/www/html/index.html
                           echo "Testing Nginx configuration..."
                           sudo -n /usr/sbin/nginx -t
                           echo "Reloading Nginx..."
                           sudo -n /usr/bin/systemctl reload nginx
                           rm -f /tmp/index.html
                           echo "Deployment completed successfully"
                          '
                    '''
                }
            }
        }

        stage('Verify Website') {
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

                        ssh \
                          -i "$SSH_KEY" \
                          -o IdentitiesOnly=yes \
                          -o StrictHostKeyChecking=yes \
                          "$SSH_USER@$DEPLOY_HOST" \
                          'sudo -n /usr/bin/systemctl is-active nginx &&
                           test -f /var/www/html/index.html &&
                           echo "Nginx is active and index.html is hosted."'
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: index.html deployed and Nginx reloaded.'
        }

        failure {
            echo 'FAILED: Check the Jenkins Console Output for the failed stage.'
        }
    }
}
