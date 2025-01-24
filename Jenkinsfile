pipeline {
    agent build-agent 

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
            steps {
                script {
                    echo "Cloning the application repository on the Master Node..."
                    git branch: 'main', url: "${APP_REPO}"
                }
            }
        }

        stage('Sync Files to Backend Server (Build-Agent)') {
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

        stage('Activate Virtual Environment (Build-Agent)') {
            steps {
                script {
                    echo "Activating virtual environment on the Build-Agent..."
                    sh """
                    ssh -i ${SSH_KEY} ${BACKEND_USER}@${BACKEND_SERVER} '
                      source ~/.bashrc
                      echo "Activating virtual environment..."
                      source venv/bin/activate
                    '
                    """
                }
            }
        }

        stage('Install Dependencies (Build-Agent)') {
            steps {
                script {
                    echo "Installing dependencies from requirements.txt on the Build-Agent..."
                    sh """
                    ssh -i ${SSH_KEY} ${BACKEND_USER}@${BACKEND_SERVER} '
                      echo "Installing dependencies from requirements.txt..."
                      source venv/bin/activate
                      pip install -r ${CHATAPP_DIR}/requirements.txt
                    '
                    """
                }
            }
        }

        stage('Run Database Migrations (Build-Agent)') {
            steps {
                script {
                    echo "Running database migrations on the Build-Agent..."
                    sh """
                    ssh -i ${SSH_KEY} ${BACKEND_USER}@${BACKEND_SERVER} '
                      echo "Running database migrations..."
                      source venv/bin/activate
                      bash /home/ShreyasChatApp/db_data.sh
                    '
                    """
                }
            }
        }

        stage('Restart Service (Build-Agent)') {
            steps {
                script {
                    echo "Restarting the ${SERVICE_NAME} service on the Build-Agent..."
                    sh """
                    ssh -i ${SSH_KEY} ${BACKEND_USER}@${BACKEND_SERVER} '
                      echo "Restarting the ${SERVICE_NAME} service..."
                      source venv/bin/activate
                      sudo systemctl restart ${SERVICE_NAME}
                    '
                    """
                }
            }
        }

        stage('Deployment Completed (Build-Agent)') {
            steps {
                script {
                    echo "Deployment tasks completed for ${BACKEND_USER}!"
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
