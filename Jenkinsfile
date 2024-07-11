pipeline {
    agent any

    environment {
        DEV_SERVER = "ec2-user@34.212.27.113"
        SSH_KEY = credentials("ssh-key-aws")
        ARTIFACT_NAME = 'python-app.tar.gz'
    }

    stages {
        stage('Build') {
            steps {
                script {
                    // Install dependencies
                    sh 'pip install -r requirements.txt'
                    // Package the application into a tarball
                    sh """
                        tar -czvf ${env.ARTIFACT_NAME} app.py requirements.txt
                    """
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                script {
                    sshagent(credentials: [${SSH_KEY}]){
                        deployToServer(DEV_SERVER)
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}

def deployToServer(server) {
    sh """
        mkdir -p ~/.ssh
        ssh-keyscan -H ${server.split('@')[1]} >> ~/.ssh/known_hosts
        scp -i ${SSH_KEY} ${ARTIFACT_NAME} ${server}:/tmp/
        ssh -i ${SSH_KEY} ${server} 'tar -xzvf /tmp/${ARTIFACT_NAME} -C /tmp/'
        ssh -i ${SSH_KEY} ${server} 'pip install -r /tmp/requirements.txt'
        ssh -i ${SSH_KEY} ${server} 'nohup python /tmp/app.py &'
    """
}

