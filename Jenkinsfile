pipeline {
    // The `agent any` directive specifies that the pipeline can run on any available agent.
    agent any

    environment {
        NETLIFY_SITE_ID = '5f2446d5-f225-4e67-9a6a-eb3b87ec2879'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Install') {
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

        stage('Tests') {
            parallel {
                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine' // Using Node.js Alpine image
                            reuseNode true
                        }
                    }
                    steps {
                        // Check if index.html exists
                        sh '''
                            echo 'Unit Tests'
                            # Debug: List contents of the build directory
                            ls -la build

                            # Check if index.html exists
                            test -f "build/index.html"

                            # Run tests
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
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
                            npx serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build
                '''
            }
        }

        stage('Approval') {
            steps {
                input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
            }
        }


        stage('Deploy prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }

        stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://legendary-mandazi-cb76b4.netlify.app'
            }

            steps {
                // Check if index.html exists
                sh '''
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}
