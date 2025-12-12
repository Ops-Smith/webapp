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
                    def dockerImage = docker.build("webapp-image:${env.BUILD_ID}")
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

                        docker run -d --name webapp-nginx -p 810:80 \
                        --restart unless-stopped webapp-image:${BUILD_ID}
                    '''
                }
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    echo "Testing site..."
                    curl -f http://localhost:810 \
                    && echo "✅ Site is live!" \
                    || (echo "⚠️ Site check failed." && exit 1)
                '''
            }
        }
    }

    post {

        success {
            echo "🎉 Deployment successful!"

            sh '''
payload=$(cat <<EOF
{
  "text": "✅ *Deployment SUCCESSFUL* for Static Webapp\\n*Build:* #${BUILD_NUMBER}\\n*Server:* http://localhost:810"
}
EOF
)

curl -X POST -H "Content-type: application/json" \
--data "$payload" "$SLACK_WEBHOOK" || true
'''
        }

        failure {
            echo "❌ Deployment failed"

            sh '''
payload=$(cat <<EOF
{
  "text": "❌ *Deployment FAILED* for Static Webapp\\n*Build:* #${BUILD_NUMBER}\\nCheck Jenkins logs for details."
}
EOF
)

curl -X POST -H "Content-type: application/json" \
--data "$payload" "$SLACK_WEBHOOK" || true
'''
        }
    }
}
