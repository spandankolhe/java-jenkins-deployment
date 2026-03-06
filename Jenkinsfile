pipeline {
    agent any

    environment {
        SSH_CREDENTIAL = 'prod-server-key'
        SERVER_IP = '13.127.231.231'
        APP_DIR = '/home/ec2-user/java-app'
        REPO_URL = 'https://github.com/spandankolhe/java-jenkins-deployment.git'
        BRANCH = 'main'
    }

    stages {

        stage('Prepare Production Server') {
            steps {
                sshagent([env.SSH_CREDENTIAL]) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@$SERVER_IP "
                        sudo dnf install java-17-amazon-corretto-devel git -y
                        mkdir -p $APP_DIR
                    "
                    '''
                }
            }
        }

        stage('Clone or Update Repository') {
            steps {
                sshagent([env.SSH_CREDENTIAL]) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@$SERVER_IP "
                        if [ -d $APP_DIR/.git ]; then
                            cd $APP_DIR
                            git pull origin $BRANCH
                        else
                            git clone -b $BRANCH $REPO_URL $APP_DIR
                        fi
                    "
                    '''
                }
            }
        }

        stage('Build and Run Application') {
            steps {
                sshagent([env.SSH_CREDENTIAL]) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@$SERVER_IP "
                        echo 'Stopping old server if running...'
                        fuser -k 8080/tcp || true

                        echo 'Building application...'
                        cd $APP_DIR
                        javac Server.java

                        echo 'Starting Java server...'
                        nohup java Server > app.log 2>&1 &
                    "
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully!"
        }
        failure {
            echo "Deployment failed!"
        }
    }
}
