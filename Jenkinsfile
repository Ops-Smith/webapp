pipeline {
    agent any

    triggers {
        githubPush()
    }

    stages {

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image using build number as tag
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
                        # Stop and remove previous container if exists
                        if docker ps -a --format '{{.Names}}' | grep -q "^webapp-nginx$"; then
                            docker stop webapp-nginx || true
                            docker rm webapp-nginx || true
                        fi

                        # Run new container on port 80
                        docker run -d --name webapp-nginx -p 80:80 --restart unless-stopped webapp-image:${BUILD_ID}
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
            slackSend(
                channel: '#devops-builds',
                tokenCredentialId: 'slack-webhook',
                message: "✅ Deployment SUCCESSFUL for Static Webapp\nBuild #${env.BUILD_NUMBER}\nServer: http://localhost"
            )
        }

        failure {
            echo "❌ Deployment failed"
            slackSend(
                channel: '#devops-builds',
                tokenCredentialId: 'slack-webhook',
                message: "❌ Deployment FAILED for Static Webapp\nBuild #${env.BUILD_NUMBER}. Check Jenkins logs."
            )
        }
    }
}
