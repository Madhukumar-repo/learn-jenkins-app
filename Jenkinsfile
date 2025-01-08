pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = '24c7d54b-977d-43e6-bafa-21d9d31223de'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "small change to check SCM polling trigger"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Test') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "we are in test stage"
                    test -f build/index.html
                    npm test
                '''
            }
        }

        stage('Deploy staging') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. SITE ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build
                '''
            }
        }

        stage('Approval') {
            steps {
                timeout(time:5, unit: 'MINUTES') {
                    input message : 'Ready to deploy.?', ok: 'Yes, i am sure i want to deploy!'  
                }
            }
        }

       stage('Deploy prod') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. SITE ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }

    }

    post{
        always{
            junit 'test-results/junit.xml'
        }
    }
}
