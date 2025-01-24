pipeline {
    agent none // No default agent; specify agents for each stage explicitly

    environment {
        APP_REPO = "https://github.com/ShreyasBhagat2802/Django_Chatapp"
        BACKEND_SERVER = "10.0.9.126"
        BACKEND_USER = "ShreyasChatApp"
        CHATAPP_DIR = "/Django_Chatapp"
        SSH_KEY = "/var/lib/jenkins/.ssh/id_rsa"
        SERVICE_NAME = "django-backend"
    }

    stages {
        stage('Clone Application Repository (Master)') {
            agent {
                label 'build-agent' // Runs on the Master Node
            }
            steps {
                script {
                    echo "Cloning the application repository on the Master Node..."
                    git branch: 'main', url: "${APP_REPO}"
                }
            }
        }

        stage('Sync Files to Backend Server (Build-Agent)') {
            agent {
                label 'build-agent' // Runs on the Slave Node
            }
            steps {
                script {
                    echo "Syncing files to the backend server from the Build-Agent..."
                    sh """
                    rsync -avz \$(pwd)/ ${BACKEND_USER}@${BACKEND_SERVER}:${CHATAPP_DIR}

                    if [ \$? -ne 0 ]; then
                      echo "ERROR: File sync failed. Please check the SSH connection and directory permissions."
                      exit 1
                    fi
                    """
                }
            }
        }

        stage('Execute Deployment Tasks on Backend Server (Build-Agent)') {
            agent {
                label 'build-agent' // Runs on the Slave Node
            }
            steps {
                script {
                    echo "Executing deployment tasks on the backend server from the Build-Agent..."
                    sh """
                    ssh -i ${SSH_KEY} ${BACKEND_USER}@${BACKEND_SERVER} '
                      set -e
                      source ~/.bashrc

                      echo "Activating virtual environment..."
                      source venv/bin/activate

                      echo "Navigating to the application directory..."
                      cd ${CHATAPP_DIR}

                      echo "Installing dependencies from requirements.txt..."
                      pip install -r requirements.txt

                      echo "Running database migrations..."
                      bash ~/db_data.sh

                      echo "Restarting the ${SERVICE_NAME} service..."
                      sudo systemctl restart ${SERVICE_NAME}

                      echo "Deployment tasks completed for ${BACKEND_USER}!"
                    '
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up temporary files, if any..."
        }
    }
}
