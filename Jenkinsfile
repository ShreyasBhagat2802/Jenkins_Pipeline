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

        stage('SSH into Backend Server (Build-Agent)') {
            agent {
                label 'build-agent' // Runs on the Slave Node
            }
            steps {
                script {
                    echo "SSH-ing into the backend server from the Build-Agent..."
                    sh """
                    ssh -i ${SSH_KEY} ${BACKEND_USER}@${BACKEND_SERVER} 'echo "Connected to the backend server!"'
                    """
                }
            }
        }

        stage('Activate Virtual Environment (Build-Agent)') {
            agent {
                label 'build-agent' // Runs on the Slave Node
            }
            steps {
                script {
                    echo "Activating virtual environment on the backend server..."
                    sh """
                    ssh -i ${SSH_KEY} ${BACKEND_USER}@${BACKEND_SERVER} '
                      set -e
                      source ~/.bashrc
                      echo "Virtual environment activated!"
                    '
                    """
                }
            }
        }

        stage('Install Dependencies (Build-Agent)') {
            agent {
                label 'build-agent' // Runs on the Slave Node
            }
            steps {
                script {
                    echo "Installing dependencies from requirements.txt..."
                    sh """
                    ssh -i ${SSH_KEY} ${BACKEND_USER}@${BACKEND_SERVER} '
                      set -e
                      cd ${CHATAPP_DIR}
                      pip install -r requirements.txt
                      echo "Dependencies installed!"
                    '
                    """
                }
            }
        }

        stage('Run Database Migrations (Build-Agent)') {
            agent {
                label 'build-agent' // Runs on the Slave Node
            }
            steps {
                script {
                    echo "Running database migrations..."
                    sh """
                    ssh -i ${SSH_KEY} ${BACKEND_USER}@${BACKEND_SERVER} '
                      set -e
                      cd ${CHATAPP_DIR}
                      bash ~/db_data.sh
                      echo "Database migrations completed!"
                    '
                    """
                }
            }
        }

        stage('Restart Service (Build-Agent)') {
            agent {
                label 'build-agent' // Runs on the Slave Node
            }
            steps {
                script {
                    echo "Restarting the ${SERVICE_NAME} service..."
                    sh """
                    ssh -i ${SSH_KEY} ${BACKEND_USER}@${BACKEND_SERVER} '
                      sudo systemctl restart ${SERVICE_NAME}
                      echo "Service restarted successfully!"
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
