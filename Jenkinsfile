pipeline {
    agent any
    
    tools {
        maven 'Maven'
    }
    
    environment {
        DOCKER_IMAGE = 'my-springboot-app'
        REPO_URL = 'https://github.com/sjmorra/MySpringBootApp.git'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Getting source code...'
                git branch: 'main', url: env.REPO_URL
            }
        }

        stage('Build Application') {
            steps {
                echo 'Building Spring Boot application...'
                sh '''
                    # Clean and build with Maven
                    mvn clean package -DskipTests

                    # Verify JAR file was created
                    ls -la target/*.jar
                    echo "✅ JAR file built successfully"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh '''
                    # Build Docker image with the newly built JAR
                    docker build -t ${DOCKER_IMAGE}:latest .

                    # List Docker images to verify
                    docker images | grep ${DOCKER_IMAGE}
                    echo "✅ Docker image built successfully"
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                echo 'Deploying Docker container...'
                sh '''
                    # Stop and remove existing container (if any)
                    docker stop my-springboot-app || true
                    docker rm my-springboot-app || true

                    # Run new container
                    docker run -d --name my-springboot-app -p 9090:9090 ${DOCKER_IMAGE}:latest

                    # Wait for container to start
                    sleep 10

                    # Verify deployment
                    if docker ps | grep my-springboot-app; then
                        echo "✅ Container deployed successfully"
                        echo "🌐 App accessible at: http://18.190.153.42:9090/news/headline"

                        # Test the endpoint
                        echo "Testing application endpoint..."
                        curl -f http://localhost:9090/news/headline || echo "⚠️ Application might still be starting up"
                    else
                        echo "❌ Deployment failed"
                        docker logs my-springboot-app
                        exit 1
                    fi
                '''
            }
        }

        stage('Cleanup') {
            steps {
                echo 'Cleaning up old Docker images...'
                sh '''
                    # Remove old/unused Docker images to save space
                    docker image prune -f

                    # Show current Docker resource usage
                    echo "Current Docker images:"
                    docker images
                    echo "Running containers:"
                    docker ps
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed!'
        }
        success {
            echo '🎉 Build and deployment successful!'
            echo 'Your Spring Boot app is running in a Docker container on EC2'
        }
        failure {
            echo '❌ Build or deployment failed'
            echo 'Check the logs above for error details'
            sh 'docker logs my-springboot-app || echo "Container not running"'
        }
    }
}