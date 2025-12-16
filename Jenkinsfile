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

        stage('SonarQube Analysis') {
            environment {
                SONAR_TOKEN = credentials('sonarqube-token')
            }
            steps {
                withSonarQubeEnv('sonar') { // Use the name you configured in Jenkins
                    sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=webapp \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=http://localhost:9000 \
                          -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    // Wait up to 10 minutes for Quality Gate to complete
                    timeout(time: 10, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Quality Gate failed: ${qg.status}"
                        }
                    }
                }
            }
        }

        stage('Push Image to Nexus') {
            environment {
                NEXUS = credentials('nexus-creds')
            }
            steps {
                sh '''
                    docker login localhost:8082 -u $NEXUS_USR -p $NEXUS_PSW

                    docker tag webapp-image:${BUILD_ID} localhost:8082/webapp-image:${BUILD_ID}
                    docker push localhost:8082/webapp-image:${BUILD_ID}
                '''
            }
        }

        stage('Deploy Dockerized NGINX') {
            steps {
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

        stage('Verify Deployment') {
            steps {
                sh '''
                    curl -f http://localhost:810 \
                    && echo "✅ Site is live!" \
                    || (echo "⚠️ Site check failed." && exit 1)
                '''
            }
        }
    }

    post {

        success {
            sh '''
payload=$(cat <<EOF
{
  "text": "✅ *Deployment SUCCESSFUL*\\nBuild: #${BUILD_NUMBER}\\nPort: 810"
}
EOF
)
curl -X POST -H "Content-type: application/json" --data "$payload" "$SLACK_WEBHOOK" || true
'''
        }

        failure {
            sh '''
payload=$(cat <<EOF
{
  "text": "❌ *Deployment FAILED*\\nBuild: #${BUILD_NUMBER}"
}
EOF
)
curl -X POST -H "Content-type: application/json" --data "$payload" "$SLACK_WEBHOOK" || true
'''
        }
    }
}
