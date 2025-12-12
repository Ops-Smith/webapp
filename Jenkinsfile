pipeline {
    agent any

    triggers {
        githubPush()
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

            withCredentials([string(credentialsId: 'slack-webhook', variable: 'WEBHOOK')]) {
                sh '''
                    curl -X POST -H "Content-type: application/json" \
                    --data "{\\"text\\": \\"✅ *Deployment SUCCESSFUL* for Static Webapp\\\\n*Build:* #${BUILD_NUMBER}\\\\n*Server:* http://localhost\\"}" \
                    "$WEBHOOK" > /dev/null 2>&1 || true
                '''
            }
        }

        failure {
            echo "❌ Deployment failed"

            withCredentials([string(credentialsId: 'slack-webhook', variable: 'WEBHOOK')]) {
                sh '''
                    curl -X POST -H "Content-type: application/json" \
                    --data "{\\"text\\": \\"❌ *Deployment FAILED* for Static Webapp\\\\n*Build:* #${BUILD_NUMBER}\\\\nCheck Jenkins logs for details.\\"}" \
                    "$WEBHOOK" > /dev/null 2>&1 || true
                '''
            }
        }
    }
}
