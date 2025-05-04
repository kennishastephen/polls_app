pipeline {
    agent any

    environment {
        EC2_USER = "ubuntu"
        EC2_HOST = "18.222.135.44"
        EC2_KEY = credentials('ec2-ssh-private-key')
        PROJECT_DIR = "/home/ubuntu/polls_app"
        VENV_DIR = "${PROJECT_DIR}/venv"
    }

    stages {
        stage('Update Code on EC2') {
            steps {
                script {
                    // Add debugging output to check the SSH connection
                    sshagent (credentials: ['ec2-ssh-private-key']) {
                        sh """
                        echo 'Connecting to EC2...'
                        ssh -v -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            echo "SSH connection successful"
                            cd ${PROJECT_DIR}
                            git pull origin master
                            python3 -m venv venv
                            ${VENV_DIR}/bin/pip install --upgrade pip
                            ${VENV_DIR}/bin/pip install -r requirements.txt
                            ${VENV_DIR}/bin/python manage.py migrate
                            ${VENV_DIR}/bin/python manage.py collectstatic --noinput
                            echo "Deployment steps completed"
                        '
                        echo 'Finished SSH commands.'
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Code updated and app restarted successfully on EC2!"
        }
        failure {
            echo "Deployment failed."
        }
    }
}
