pipeline {
    agent any

    stages {
        stage('Unit Tests') {
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
                }
            }
        }

    //     stage('Build') {
    //         // This is a comment
    //         /*
    //         This is also a comment
    //         */
    //         agent {
    //             docker {
    //                 image 'node:18-alpine'
    //                 reuseNode true
    //             }
    //         }
    //         steps {
    //             sh '''
    //                 echo "-----------------BUILD START-----------------"
    //                 ls -la
    //                 node --version
    //                 npm --version
    //                 npm ci
    //                 npm run build
    //                 ls -la
    //                 echo "-----------------TEST COMPLETED-----------------"
    //             '''
    //         }
    // // }
    // }

        post {
            always {
                junit 'jest-results/junit.xml'

                publishHTML([
                allowMissing: false,
                alwaysLinkToLastBuild: false,
                icon: '',
                keepAll: false,
                reportDir: 'playwright-report',
                reportFiles: 'index.html',
                reportName: 'Playwright HTML Report',
                reportTitles: '',
                useWrapperFileDirectly: true
            ])

                echo '"-----------------Pipeline completed-----------------'
            }
        }
    }
}
