pipeline {
    agent any

    stages {
        stage('Build') {
            // This is a comment
            /*
            This is also a comment
            */
            agent {
                    docker {
                        image  'node:18-alpine'
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
        stage('Test') {
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
                node_modules/.bin/serve
                npx playwright test
                echo "-------------E2E COMPLETE-----------------"
                '''
            }
        }
    }
    post {
        always {
            echo '"-----------------Pipeline completed-----------------'
            junit 'test-results/junit.xml'
        }
    }
}
