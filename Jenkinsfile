pipeline{
    agent any
    triggers {
        pollSCM('H/1 * * * *') // Check repo every 2 minutes
    }
    stages{
        stage("Display a greeting message"){
            steps{
                sh'''
                    echo "Hello David! This is how you deploy a static webapp on NGINX using Jenkins"
                '''
            }
        }
        stage("Clone Repo and deploy using NGINX"){
            steps{
                sh'''
                    sudo apt update
                    sudo systemctl enable nginx
                    sudo systemctl start nginx
                    sudo systemctl status nginx
                    sudo chown -R root:root /var
                    cd /var/www
                    sudo rm -rf html/
                    sudo mkdir html
                    sudo git clone https://github.com/Ops-Smith/webapp.git ./html
                    sudo systemctl restart nginx
                '''

            }
        }
    }
    post{
        success{
            echo "Deployment successful! Visit http://localhost"
        }
    }
}