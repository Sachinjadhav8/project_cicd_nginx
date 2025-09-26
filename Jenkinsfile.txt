pipeline {
    agent any

    stages {
        stage('checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Sachinjadhav8/project_cicd_nginx.git']])
            }
        }
        stage('deployment') {
            environment {
                EC2_USER = 'ubuntu'                   // EC2 username
                EC2_HOST = '13.233.64.113'       // EC2 public IP
                SSH_KEY = credentials('credentials') // Secret file
                INDEX_FILE = 'index.html'             // File to deploy
            }
                steps {
                    sh """
                    # Copy file from Jenkins to EC2
                    scp -i \$SSH_KEY -o StrictHostKeyChecking=no ${WORKSPACE}/${INDEX_FILE} ${EC2_USER}@${EC2_HOST}:/tmp/${INDEX_FILE}

                    # Connect to EC2 and deploy
                    ssh -i \$SSH_KEY -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "
                        cd /var/www/html
                        if [ -f index.html ]; then sudo mv index.html index.html.bak; fi
                        sudo mv /tmp/${INDEX_FILE} .
                        sudo chown www-data:www-data index.html
                        echo "index.html updated successfully!"
                    "
                    """
                
                }
        }
    }
}
