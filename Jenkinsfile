
pipeline {
    agent any

    stages {
        stage('Test EC2 Credential') {
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
                        test -f "$SSH_KEY"
                        echo "SSH private key loaded successfully"
                        echo "SSH username: $SSH_USER"
                    '''
                }
            }
        }
    }
}
