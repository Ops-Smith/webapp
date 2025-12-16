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
                    docker.build("webapp-image:${BUILD_ID}")
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-local') {
                    sh '''
                      sonar-scanner \
                        -Dsonar.projectKey=webapp \
                        -Dsonar.sources=.
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Push Image to Nexus') {
            environment {
                NEXUS = credentials('nexus-creds')
            }
            steps {
                sh '''
                  docker login localhost:8082 \
                    -u $NEXUS_USR \
                    -p $NEXUS_PSW

                  docker tag webapp-image:${BUILD_ID} \
                    localhost:8082/webapp-image:${BUILD_ID}

                  docker push localhost:8082/webapp-image:${BUILD_ID}
                '''
            }
        }

        stage('Deploy Dockerized NGINX') {
            steps {
                sh '''
                  docker rm -f webapp-nginx || true

                  docker run -d \
                    --name webapp-nginx \
                    -p 810:80 \
                    --restart unless-stopped \
                    localhost:8082/webapp-image:${BUILD_ID}
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                  curl -f http://localhost:810 \
                  && echo "✅ App is running" \
                  || exit 1
                '''
            }
        }
    }

    post {
        success {
            sh '''
curl -X POST -H "Content-type: application/json" \
--data "{\"text\":\"✅ Deployment SUCCESSFUL\\nBuild #${BUILD_NUMBER}\"}" \
"$SLACK_WEBHOOK" || true
'''
        }

        failure {
            sh '''
curl -X POST -H "Content-type: application/json" \
--data "{\"text\":\"❌ Deployment FAILED\\nBuild #${BUILD_NUMBER}\"}" \
"$SLACK_WEBHOOK" || true
'''
        }
    }
}
