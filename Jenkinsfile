pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID = 'c92a493c-e774-47d0-8b8a-3760ae4b0d07'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages{
        stage('AWS'){
            agent{
                docker{
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                }
            }
            steps{
                sh '''
                    aws --version
                '''
            }
        }
        stage('Build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        
        stage('parallel'){
            parallel{
                stage('Test') {
                    agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "test sucsseded"
                            test -f build/index.html
                            npm test
                            echo "test sucsseded210"
                            ls -la build
                        '''
                    }
                }
                stage('e2e') {
                    agent{
                        docker{
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                }
            }                
            
        }


        stage('deployment staging') {
            agent{
                docker{
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = "to be set"
            }
            steps {
                sh '''
                    netlify --version
                    echo "deploying to staging, siteid: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deply-output.json
                    CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deply-output.json)
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }


        stage('deployment prod') {
            agent{
                docker{
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = 'https://effortless-pony-287603.netlify.app'
            }
            steps {
                sh '''
                    node --version
                    netlify --version
                    echo "deploying to prod, siteid: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }        
    
    
    }
    post{
        always{
            junit 'jest-results/junit.xml'
        }
    }
    
}

