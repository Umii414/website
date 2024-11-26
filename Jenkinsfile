pipeline {
    agent any
    environment {
        GIT_REPO = 'https://github.com/Umii414/website.git'
        DOCKER_CREDENTIALS = 'docker-credentials'  // The ID of Docker credentials in Jenkins
    }
    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout the repository code
                    checkout scm
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    echo "Building the product..."
                    // Example of building a Docker image
                    sh 'docker build -t myapp .'
                }
            }
        }
        stage('Publish') {
            when {
                branch 'master' // Execute only for master branch
            }
            steps {
                script {
                    try {
                        // Docker Login (if pushing images)
                        echo 'Logging into Docker registry...'
                        docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS) {
                            // Stop and remove any existing container with the same name
                            echo 'Cleaning up old containers...'
                            sh '''
                            docker stop website-container || true
                            docker rm website-container || true
                            '''
                            
                            // Run Docker container with Ubuntu base
                            echo 'Running Docker container...'
                            sh '''
                            docker run -d -p 82:80 --name website-container ubuntu:latest
                            '''
                            
                            // Install Apache in the container
                            echo 'Installing Apache in the container...'
                            sh '''
                            docker exec website-container bash -c "apt-get update && apt-get install -y apache2"
                            '''
                            
                            // Copy website files into the container
                            echo 'Copying website files...'
                            sh '''
                            docker cp . website-container:/var/www/html
                            '''
                            
                            // Restart the container
                            echo 'Restarting Docker container...'
                            sh '''
                            docker restart website-container
                            '''
                        }
                    } catch (Exception e) {
                        echo "An error occurred: ${e.message}"
                        error("Pipeline failed due to an error in the Publish stage.")
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'CI/CD process finished'
        }
    }
}
