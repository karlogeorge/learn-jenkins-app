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
                echo "--------TEST STAGE-----------"
                test -f build/index.html
                npm test
                echo "-----------------TEST COMPLETED SUCCESSFULLY--------------"
                '''
            }
        }
        stage('E2E') {
            agent {
                    docker {
                        image 'docker pull mcr.microsoft.com/playwright:v1.60.0-noble'
                        reuseNode true
                    }
            }
            steps {
                sh '''
                echo "-------------Starting E2E-----------------"
                npm install -g serve
                npx playwright test
                echo "-------------END OF E2E-----------------"
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
