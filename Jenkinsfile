pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        SLACK_WEBHOOK = credentials('slack-webhook')
    }

    stages {

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("webapp-image:${env.BUILD_ID}")
                }
            }
        }

        stage('Display Greeting') {
            steps {
                echo "🚀 Deploying to PRODUCTION"
                sh 'echo "Hello David! Deploying static webapp via Docker NGINX"'
            }
        }

        stage('Deploy Dockerized NGINX') {
            steps {
                script {
                    sh '''
                        if docker ps -a --format '{{.Names}}' | grep -q "^webapp-nginx$"; then
                            docker stop webapp-nginx || true
                            docker rm webapp-nginx || true
                        fi

                        docker run -d --name webapp-nginx -p 800:80 \
                        --restart unless-stopped webapp-image:${BUILD_ID}
                    '''
                }
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    echo "Testing website..."
                    curl -f http://localhost && echo "✅ Site is live!" || (echo "⚠️ Site check failed." && exit 1)
                '''
            }
        }
    }

    post {
        success {
            echo "🎉 Deployment successful! Visit: http://localhost"

            sh """
            curl -X POST -H 'Content-type: application/json' \
            --data '{
              "text": "✅ *Deployment SUCCESSFUL* for Static Webapp\n*Build:* #${env.BUILD_NUMBER}\n*Server:* http://localhost"
            }' $SLACK_WEBHOOK
            """
        }

        failure {
            echo "❌ Deployment failed"

            sh """
            curl -X POST -H 'Content-type: application/json' \
            --data '{
              "text": "❌ *Deployment FAILED* for Static Webapp\n*Build:* #${env.BUILD_NUMBER}\nCheck Jenkins logs for details."
            }' $SLACK_WEBHOOK
            """
        }
    }
}
pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        SLACK_WEBHOOK = credentials('slack-webhook')
    }

    stages {

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("webapp-image:${env.BUILD_ID}")
                }
            }
        }

        stage('Display Greeting') {
            steps {
                echo "🚀 Deploying to PRODUCTION"
                sh 'echo "Hello David! Deploying static webapp via Docker NGINX"'
            }
        }

        stage('Deploy Dockerized NGINX') {
            steps {
                script {
                    sh '''
                        if docker ps -a --format '{{.Names}}' | grep -q "^webapp-nginx$"; then
                            docker stop webapp-nginx || true
                            docker rm webapp-nginx || true
                        fi

                        docker run -d --name webapp-nginx -p 800:80 \
                        --restart unless-stopped webapp-image:${BUILD_ID}
                    '''
                }
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    echo "Testing site on port 800..."
                    curl -f http://localhost:800 && echo "✅ Site is live!" || (echo "⚠️ Site check failed." && exit 1)
                '''
            }
        }
    }

    post {
        success {
            echo "🎉 Deployment successful! Visit: http://localhost:800"

            sh """
            curl -X POST -H 'Content-type: application/json' \
            --data '{"text": "✅ *Deployment SUCCESSFUL* for Static Webapp\n*Build:* #${BUILD_NUMBER}\n*Server:* http://localhost:800"}' \
            "$SLACK_WEBHOOK"
            """
        }

        failure {
            echo "❌ Deployment failed"

            sh """
            curl -X POST -H 'Content-type: application/json' \
            --data '{"text": "❌ *Deployment FAILED* for Static Webapp\n*Build:* #${BUILD_NUMBER}\nCheck Jenkins logs for details."}' \
            "$SLACK_WEBHOOK"
            """
        }
    }
}
