pipeline {
    // The `agent any` directive specifies that the pipeline can run on any available agent.
    agent any

    stages {
        stage('Install'){
            agent {
                docker {
                    image 'node:18-alpine' // Using Node.js Alpine image
                    reuseNode true
                }
            }
            steps {
                sh '''
                    # List all files in the current workspace for debugging purposes.
                    ls -la
                    
                    # Display the Node.js version for compatibility confirmation.
                    node --version
                    
                    # Display the npm version for compatibility confirmation.
                    npm --version
                    
                    # Install project dependencies using npm's clean install.
                    npm ci
                '''
            }
        }

        stage('Build') {
            // Specify the Docker container that will be used for this stage.
            agent {
                docker {
                    // Use the Node.js 18 Alpine Docker image for this stage.
                    image 'node:18-alpine'
                    
                    // Reuse the workspace of the Jenkins agent where the container runs.
                    reuseNode true
                }
            }
            steps {
                // Execute shell commands to build the Node.js application.
                sh '''
                    # Build the application using the npm `build` script defined in package.json.
                    npm run build
                    
                    # List all files after the build process to confirm the output directory.
                    ls -la
                '''
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine' // Using Node.js Alpine image
                    reuseNode true
                }
            }
            steps {
                // Check if index.html exists
                sh '''
                    # Debug: List contents of the build directory
                    ls -la build

                    # Check if index.html exists
                    test -f "build/index.html"

                    # Run tests
                    npm test
                '''
            }
        }

        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps {
                // Check if index.html exists
                sh '''
                    npm install serve --save-dev
                    npx serve -s build
                    npx playwright test
                '''
            }
        }
    }
    
    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}
