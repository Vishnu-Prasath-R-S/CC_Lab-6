pipeline {
    agent any
    stages {
        stage('Build Backend Image') {
            steps {
                sh '''
                # Remove old image to ensure a fresh build
                docker rmi -f backend-app || true
                
                # Build using the 'backend' folder at the root of the repo
                docker build -t backend-app backend
                '''
            }
        }
        stage('Deploy Backend Containers') {
            steps {
                sh '''
                # Create the network if it doesn't exist
                docker network create app-network || true
                
                # Remove any existing backend containers
                docker rm -f backend1 backend2 || true
                
                # Start two backend instances on the shared network
                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app
                '''
            }
        }
        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                # Remove existing Nginx container
                docker rm -f nginx-lb || true
                
                # IMPORTANT: Wait for backend DNS to register in the Docker network
                sleep 10
                
                # Start Nginx and map to host port 80
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx
                
                # Copy the configuration and reload
                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }
    post {
        success {
            echo 'Pipeline executed successfully. NGINX load balancer is running at http://localhost:80'
        }
        failure {
            echo 'Pipeline failed. Please check the console output for errors.'
        }
    }
}
