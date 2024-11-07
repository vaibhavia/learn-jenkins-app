pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
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
                    echo "npm dependencies installed using npm ci command and then we ran the build"
                '''
             }
        }
        stage('Test'){
            steps{
                sh '''
                echo 'Test stage'
                test -f build/index.html
                npm test
                '''
            }
        }
    }
}
