pipeline {
    // The `agent any` directive specifies that the pipeline can run on any available agent.
    agent any

    environment {
        NETLIFY_SITE_ID = '5f2446d5-f225-4e67-9a6a-eb3b87ec2879'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }
    stages {
        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-s3', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        aws s3 ls
                    '''
                }
            }
        }
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
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage('Deploy staging') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
            }

            steps {
                // Check if index.html exists
                sh '''
                    netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json

                    # Assign the output to the shell variable CI_ENVIRONMENT_URL
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                    export CI_ENVIRONMENT_URL

                    # Echo the value to confirm it
                    echo "CI_ENVIRONMENT_URL is now: $CI_ENVIRONMENT_URL"

                    # Run tests
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Staging', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Deploy prod') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://legendary-mandazi-cb76b4.netlify.app'
            }

            steps {
                // Check if index.html exists
                sh '''
                    node --version
                    npm install netlify-cli
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Prod', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}
