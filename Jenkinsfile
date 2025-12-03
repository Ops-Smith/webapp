pipeline {
    agent any
    triggers {
        githubPush()
    }
    stages {
        stage('Display Greeting') {
            steps {
                echo "🚀 Deploying to PRODUCTION"
                sh '''
                    echo "Hello David! Deploying static webapp to NGINX"
                '''
            }
        }
        stage('Deploy to NGINX') {
            steps {
                sh '''
                    # Ensure NGINX is running
                    sudo systemctl start nginx || true
                    
                    # Change directory
                    cd /var/www

                    # Clear html directory
                    sudo rm -rf html

                    # Create new directory
                    sudo mkdir html
                    
                    # Clone latest code
                    sudo git clone https://github.com/Ops-Smith/webapp.git ./html
                    
                    # Set proper permissions
                    sudo chown -R www-data:www-data /var/www/html
                    
                    # Reload NGINX
                    sudo systemctl reload nginx
                    echo "✅ Production site deployed!"
                '''
            }
        }
        stage('Verify') {
            steps {
                sh '''
                    echo "Testing website..."
                    curl -f http://localhost && echo "✅ Site is live!" || echo "⚠️ Site check failed. Check"
                '''
            }
        }
    }
    post {
        success {
            echo "🎉 Deployment successful! Visit: http://localhost"
        }
        failure {
            echo "❌ Deployment failed"
        }
    }
}