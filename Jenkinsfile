pipeline {
    agent { label 'laravel-agent' }

    environment {
        DEPLOY_PATH = '/var/www/your_name'
        DEPLOY_HOST = '178.128.93.188'

        // Jenkins credentials IDs
        PROD_ENV = credentials('laravel-env-prod')
        DEPLOY_KEY = credentials('deploy-server-ssh')
        TELEGRAM_BOT_TOKEN = credentials('telegram-bot-token')
        TELEGRAM_CHAT_ID = credentials('telegram-chat-id')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Prepare Env') {
            steps {
                sh '''
                    cp "$PROD_ENV" .env.production
                '''
            }
        }

        stage('Build Laravel') {
            steps {
                sh '''
                    set -e

                    composer install --no-interaction --prefer-dist --optimize-autoloader

                    cp .env.example .env
                    php artisan key:generate --force

                    npm install
                    npm run build

                    tar \
                      --exclude=.git \
                      --exclude=node_modules \
                      --exclude=.env \
                      --exclude=.env.production \
                      --exclude=Jenkinsfile \
                      --exclude=ansible \
                      -czf release.tar.gz .
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    set -e
                    chmod 600 "$DEPLOY_KEY"

                    ansible-playbook \
                      -i ansible/inventory.ini \
                      ansible/deploy.yml \
                      -u "$DEPLOY_KEY_USR" \
                      --private-key "$DEPLOY_KEY" \
                      --extra-vars "deploy_path=$DEPLOY_PATH workspace=$WORKSPACE"
                '''
            }
        }
    }

    post {
        failure {
            sh '''
                set +x
                curl -s -X POST "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage" \
                  -d chat_id="$TELEGRAM_CHAT_ID" \
                  --data-urlencode text="❌ Jenkins failed: $JOB_NAME #$BUILD_NUMBER - $BUILD_URL"
            '''
        }
    }
}
