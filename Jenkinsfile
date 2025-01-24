pipeline {
    agent any

    environment {
        APP_REPO = "https://github.com/ShreyasBhagat2802/Django_Chatapp"
        BACKEND_SERVER = "10.0.9.126"
        BACKEND_USER = "ShreyasChatApp"
        CHATAPP_DIR = "/Django_Chatapp"
        SSH_KEY = "/var/lib/jenkins/.ssh/id_rsa"
        SERVICE_NAME = "django-backend"
    }

    stages {
        stage('Clone Application Repository') {
            steps {
                script {
                    echo "Cloning the application repository..."
                    git branch: 'main', url: "${APP_REPO}"
                }
            }
        }

        stage('Sync Files to Backend Server') {
            steps {
                script {
                    echo "Syncing files to the backend server..."
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

        stage('Activate Virtual Environment') {
            steps {
                script {
                    echo "Activating virtual environment on the backend server..."
                    sh """
                    ssh -i ${SSH_KEY} ${BACKEND_USER}@${BACKEND_SERVER} '
                      set -e
                      source ~/.bashrc
                      source venv/bin/activate
                    '
                    """
                }
            }
        }

        stage('Navigate to Application Directory') {
            steps {
                script {
                    echo "Navigating to the application directory on the backend server..."
                    sh """
                    ssh -i ${SSH_KEY} ${BACKEND_USER}@${BACKEND_SERVER} '
                      cd ${CHATAPP_DIR}
                    '
                    """
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    echo "Installing dependencies from requirements.txt on the backend server..."
                    sh """
                    ssh -i ${SSH_KEY} ${BACKEND_USER}@${BACKEND_SERVER} '
                      cd ${CHATAPP_DIR}
                      pip install -r requirements.txt
                    '
                    """
                }
            }
        }

        stage('Run Database Migrations') {
            steps {
                script {
                    echo "Running database migrations on the backend server..."
                    sh """
                    ssh -i ${SSH_KEY} ${BACKEND_USER}@${BACKEND_SERVER} '
                      source ~/.bashrc
                      bash ~/db_data.sh
                    '
                    """
                }
            }
        }

        stage('Restart Backend Service') {
            steps {
                script {
                    echo "Restarting the ${SERVICE_NAME} service on the backend server..."
                    sh """
                    ssh -i ${SSH_KEY} ${BACKEND_USER}@${BACKEND_SERVER} '
                      sudo systemctl restart ${SERVICE_NAME}
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
