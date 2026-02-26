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
                docker network create app-network || true

                # remove old containers safely
                docker rm -f backend1 backend2 2>/dev/null || true

                # start backend containers
                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app

                echo "=== Running containers ==="
                docker ps
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                docker rm -f nginx-lb || true

                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx

                # wait for nginx to fully start
                sleep 6

                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf

                # reload safely
                docker exec nginx-lb nginx -s reload || true

                echo "=== Final container status ==="
                docker ps
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