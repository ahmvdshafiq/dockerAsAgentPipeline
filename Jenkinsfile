pipeline {
    agent any

    environment{
        dockerHub_creds_id = '1c9e152e-526e-4ee6-a18d-abb0c7f5dac0'
        docker_image = 'madbakoyoko/dockerpipelinehelloworld'
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:16-alpine'
                    args '-v /app/node_modules'
                }
            }
            steps {
                sh 'npm install'
            }
        }
        

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${docker_image} ."
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', dockerHub_creds_id) {
                      docker.image(${docker_image}).push()
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Use SSH to deploy the app on the EC2 instance
                    sh """
                    ssh -o StrictHostKeyChecking=no -i /home/ahmad/shafiqRonaldoKey.pem ubuntu@13.215.51.100 <<EOF
                        # Stop any running container of the same app
                        docker stop node-app || true
                        docker rm node-app || true
                        
                        # Pull the latest Docker image
                        docker pull ${docker_image}:${env.BUILD_NUMBER}
                        
                        # Run the container
                        docker run -d -p 3000:3000 --name node-app ${docker_image}:${env.BUILD_NUMBER}
                    EOF
                    """
                }   
            }
        }
    }
}
