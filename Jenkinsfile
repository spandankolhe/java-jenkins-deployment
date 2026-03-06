pipeline {
    agent any

    environment {
        SSH_CREDENTIAL = 'prod-server-key'
        SERVER_IP = '13.127.231.231'
        APP_DIR = '/home/ec2-user/java-app'
    }

    stages {

        stage('Clone Repository') {
            steps {
                echo "Cloning GitHub repository"
                git 'https://github.com/spandankolhe/java-jenkins-deployment.git'
            }
        }

        stage('Prepare Server') {
            steps {
                sshagent([env.SSH_CREDENTIAL]) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@$SERVER_IP "
                        sudo dnf install java-17-amazon-corretto -y
                        mkdir -p $APP_DIR
                    "
                    '''
                }
            }
        }

        stage('Copy Application') {
            steps {
                sshagent([env.SSH_CREDENTIAL]) {
                    sh '''
                    scp -o StrictHostKeyChecking=no Server.java ec2-user@$SERVER_IP:$APP_DIR/
                    '''
                }
            }
        }

        stage('Deploy Application') {
            steps {
                sshagent([env.SSH_CREDENTIAL]) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@$SERVER_IP "
                        cd $APP_DIR
                        pkill -f 'java Server' || true
                        javac Server.java
                        nohup java Server > app.log 2>&1 &
                    "
                    '''
                }
            }
        }

    }
}