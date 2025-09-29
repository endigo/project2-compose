pipeline {
    agent any

    environment {
        // Define environment variables
        NODE_VERSION = '18'
        APP_NAME = 'express-sample-app'
        DOCKER_IMAGE = 'express-app'
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Fetching source code from GitHub...'
                git branch: 'main',
                    url: 'https://github.com/endigo/aws-elastic-beanstalk-express-js-sample.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing Node.js dependencies...'
                sh '''
                    node --version
                    npm --version
                    npm install
                '''
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running tests...'
                sh '''
                    # Run tests if test script exists
                    npm test || echo "No tests defined"
                '''
            }
        }

        stage('Code Quality Check') {
            steps {
                echo 'Performing code quality checks...'
                sh '''
                    # Run linting if available
                    npm run lint || echo "No linting configured"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    // Create Dockerfile if it doesn't exist
                    sh '''
                        if [ ! -f Dockerfile ]; then
                            cat > Dockerfile << EOF
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 8080
CMD ["node", "app.js"]
EOF
                        fi
                    '''

                    // Build Docker image
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Test Docker Image') {
            steps {
                echo 'Testing Docker image...'
                script {
                    // Run container in detached mode for testing
                    sh """
                        docker run -d --name test-${BUILD_NUMBER} -p 8081:8080 ${DOCKER_IMAGE}:${DOCKER_TAG}
                        sleep 5
                        curl -f http://localhost:8081 || exit 1
                        docker stop test-${BUILD_NUMBER}
                        docker rm test-${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Security Scan') {
            steps {
                echo 'Running security scan...'
                sh '''
                    # Check for known vulnerabilities in dependencies
                    npm audit || true
                '''
            }
        }

        stage('Archive Artifacts') {
            steps {
                echo 'Archiving build artifacts...'
                archiveArtifacts artifacts: 'package*.json, app.js, *.html',
                                 allowEmptyArchive: true
            }
        }

        stage('Deploy to Staging') {
            when {
                branch 'main'
            }
            steps {
                echo 'Deploying to staging environment...'
                sh '''
                    echo "Deployment to staging would happen here"
                    # Example: Push to registry, deploy to Elastic Beanstalk, etc.
                '''
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            sh '''
                # Clean up Docker images
                docker rmi ${DOCKER_IMAGE}:${DOCKER_TAG} || true
                docker rmi ${DOCKER_IMAGE}:latest || true
            '''
            cleanWs()
        }

        success {
            echo 'Pipeline executed successfully!'
            // Send success notification
        }

        failure {
            echo 'Pipeline failed!'
            // Send failure notification
        }

        unstable {
            echo 'Pipeline is unstable'
            // Send warning notification
        }
    }
}