pipeline {
    agent any

    environment {
        APP_REPO = "https://github.com/ShreyasBhagat2802/Django_Chatapp"
        DEPLOY_REPO = "https://github.com/ShreyasBhagat2802/Jenkins_Pipeline"
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

        stage('Execute Deployment Tasks') {
            steps {
                script {
                    echo "Starting deployment process..."

                    // Syncing files to the backend server
                    echo "Syncing files to the backend server..."
                    sh """
                    rsync -avz \$(pwd)/ ${BACKEND_USER}@${BACKEND_SERVER}:${CHATAPP_DIR}

                    if [ \$? -ne 0 ]; then
                      echo "ERROR: File sync failed. Please check the SSH connection and directory permissions."
                      exit 1
                    fi
                    """

                    // SSH into the backend server to execute deployment tasks
                    echo "Executing tasks on the backend server as ${BACKEND_USER}..."
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
                      if [ -f ~/db_data.sh ]; then
                        bash ~/db_data.sh
                      else
                        echo "WARNING: db_data.sh not found. Skipping database migration tasks."
                      fi

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
