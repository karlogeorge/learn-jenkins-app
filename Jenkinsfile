pipeline {
    agent any

    environment {
        // NETLIFY_SITE_ID = '84d22339-971c-45ae-bfcf-1a7e1a38fe24'
        NETLIFY_SITE_ID = credentials('netlify-site-id')
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = '1.2.3'
    }

    stages {
        stage('Build') {
            // This is a comment
            /*
            This is also a comment
            */
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "-----------------BUILD START-----------------"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                    echo "-----------------TEST COMPLETED-----------------"
                '''
            }
        }

        stage('TESTS') {
            parallel {
                stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "-----------------TEST START-----------------"
                            test -f build/index.html
                            npm test
                            echo "-----------------TEST COMPLETED SUCCESSFULLY--------------"
                        '''
                    }

                    post {
                        always {
                            junit 'jest-results/junit.xml'
                            echo '"-----------------Pipeline TEST completed-----------------'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.60.0-noble'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "-------------E2E START-----------------"
                            npm install serve

                            # This will start this command in background
                            node_modules/.bin/serve -s build &
                            sleep 15
                            npx playwright test --reporter=html
                            echo "-------------E2E COMPLETE-----------------"
                        '''
                    }

                    post {
                        always {
                            publishHTML([
                            allowMissing: false,
                            alwaysLinkToLastBuild: false,
                            icon: '',
                            keepAll: false,
                            reportDir: 'playwright-report',
                            reportFiles: 'index.html',
                            reportName: 'Playwright E2E Local',
                            reportTitles: '',
                            useWrapperFileDirectly: true
                            ])
                            echo '"-----------------Pipeline E2E completed-----------------'
                        }
                    }
                }
            }
        }
        stage('Deploy Staging') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.60.0-noble'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
            }
            steps {
                sh '''
                    echo "-------------Staging Deploy START-----------------"
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    echo "---------Deploying to Staging : $NETLIFY_SITE_ID --------"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --no-build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)
                    npx playwright test --reporter=html
                    echo "-------------Staging Deploy COMPLETE-----------------"
                '''
            }

            post {
                always {
                    publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    icon: '',
                    keepAll: false,
                    reportDir: 'playwright-report',
                    reportFiles: 'index.html',
                    reportName: 'Staging E2E',
                    reportTitles: '',
                    useWrapperFileDirectly: true
                    ])
                    echo '"-----------------Staging Pipeline E2E completed-----------------'
                }
            }
        }
        // stage('Approval') {
        //     steps {
        //         timeout(time: 15, unit: 'MINUTES') {
        //             input message: 'Ready to Deploy to PROD ?',
        //             ok: 'Yes, I am ready to deploy.'
        //         }
        //     }
        // }
        stage('Deploy PROD') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.60.0-noble'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://vocal-sorbet-197979.netlify.app/'
            }
            steps {
                sh '''
                    echo "-------------PROD E2E START-----------------"
                    node --version
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "---------Deploying to prod id : $NETLIFY_SITE_ID --------"
                    node_modules/.bin/netlify status
                    echo "-----------------------DEPLOY START---------------------"
                    node_modules/.bin/netlify deploy --dir=build --prod --no-build 
                    echo "----------------------DEPLOY COMPLETED------------------"
                    npx playwright test --reporter=html
                    echo "-------------PROD E2E COMPLETE-----------------"
                '''
            }

            post {
                always {
                    publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    icon: '',
                    keepAll: false,
                    reportDir: 'playwright-report',
                    reportFiles: 'index.html',
                    reportName: 'Playwright E2E Prod',
                    reportTitles: '',
                    useWrapperFileDirectly: true
                    ])
                    echo '"-----------------PROD Pipeline E2E completed-----------------'
                }
            }
        }
    }
}
