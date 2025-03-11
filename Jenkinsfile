pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = 'f1664910-c877-4b59-9b70-f9598dc4dbdb'
        NETLIFY_AUTH_TOKEN = credentials('netlify_token')
    }
    stages {
        stage('Build') {
            agent { 
                docker {
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

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent { 
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }    
                    steps {
                        sh '''
                            test -f build/index.html
                            npm test
                        '''
                    }
                }
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
                sh '''
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test --reporter=html
                '''    
            } 
        }

        stage('Deploy') {  
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
                '''
            }
        }
    }

    post {
        always {
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'path/to/report', reportFiles: 'index.html', reportName: 'HTML Report'])
        }
    }
}
