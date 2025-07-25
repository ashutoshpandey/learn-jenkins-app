pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '744b0a44-6aa6-4285-9921-453a54420eaa'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
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

        stage('Test') {
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

        stage ('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.54.1-noble'
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

            post {
                always {
                    junit 'jest-results/junit.xml'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
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
                    .nde_modules/.bin/netlify --version
                    echo "Deploying to production with site id $NETLIFY_SITE_ID
                    .nde_modules/.bin/netlify status
                    .nde_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
    }
}
