pipeline {
    agent any

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                # Stop and remove ALL related containers first
                docker rm -f nginx-lb backend1 backend2 || true

                # Remove old network safely
                docker network rm app-network || true

                # Create fresh network
                docker network create app-network

                # Start backend containers
                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app

                sleep 3
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx

                sleep 2

                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully. NGINX load balancer is running.'
        }
        failure {
            echo 'Pipeline failed. Check console logs for errors.'
        }
    }
}
