pipeline {
    agent { label 'laravel-agent' }

    environment {
        DEPLOY_PATH = '/var/www/sophat_odom'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Laravel') {
            steps {
                sh '''
                    set -e

                    cp .env.example .env

                    php -v
                    composer --version
                    node -v
                    npm -v
                    ansible --version

                    composer install --no-interaction --prefer-dist --optimize-autoloader
                    npm install
                    npm run build

                    tar \
                      --exclude=.git \
                      --exclude=node_modules \
                      --exclude=Jenkinsfile \
                      --exclude=ansible \
                      -czf release.tar.gz .
                '''
            }
        }

        stage('Deploy with Ansible') {
            steps {
                sh '''
                    ansible-playbook \
                      -i ansible/inventory.ini \
                      ansible/deploy.yml \
                      --extra-vars "deploy_path=$DEPLOY_PATH workspace=$WORKSPACE"
                '''
            }
        }
    }

    post {
        failure {
            echo 'Build failed'
        }
    }
}
