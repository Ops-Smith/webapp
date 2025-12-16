pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        SLACK_WEBHOOK = credentials('slack-webhook')
        SONAR_TOKEN   = credentials('sonarqube-token')
        NEXUS         = credentials('nexus-creds')
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
            steps {
                script {
                    // Use Jenkins-managed SonarQube Scanner
                    def scannerHome = tool name: 'sonar-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'

                    withSonarQubeEnv('sonar') {
                        sh """
                            $scannerHome/bin/sonar-scanner \
                                -Dsonar.projectKey=webapp \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=$SONAR_HOST_URL \
                                -Dsonar.login=$SONAR_TOKEN
                        """
                    }
                }
            }
        }

        stage('Push Image to Nexus') {
            steps {
                sh """
                    echo $NEXUS_PSW | docker login http://localhost:8081 -u $NEXUS_USR --password-stdin
                    docker tag webapp-image:${BUILD_ID} localhost:8081/docker-hosted/webapp-image:${BUILD_ID}
                    docker push localhost:8081/docker-hosted/webapp-image:${BUILD_ID}
                """
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
