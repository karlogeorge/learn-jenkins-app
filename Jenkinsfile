pipeline {
    agent any

    stages {
        stage('Hello') {
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
    }
    post {
        always {
            echo 'Pipeline completed'
            junit 'test-results/junit.xml'
        }
    }
}
