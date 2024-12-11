pipeline {
    // The `agent any` directive specifies that the pipeline can run on any available agent.
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DEFAULT_REGION = 'eu-west-2'
    }
    stages {
        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli:2.22.14'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-s3', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        yum install jq -y
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                        echo $LATEST_TD_REVISION
                        aws ecs update-service --cluster LearnJenkinsApp-Cluster-Prod --service LearnJenkinsApp-Service-Prod --task-definition LearnJenkinsApp-TaskDefinition-Prod:$LATEST_TD_REVISION
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
    }
}
