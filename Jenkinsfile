pipeline {
    agent any

    stages {
        /* 
          with this format you can comment multiple lines...Called comment block format.
          If we directly want to run the TEST stage, we can comment the BUILD stage block and thus only TEST stage will be executed.
        */
        // This format is used to comment single line. Commented lines turns green.
        /*
        stage('Build') {
            agent {
                docker{
                    image 'node:18-alpine'
                    //here docker will use node image to run node.js. format says node image colon version name dash slim linux sytem type perfect for ci cd i.e. is alpine
                    reuseNode true
                    //this flag will use the same workspace for stages using docker as well as not using docker. default flag is false and when false seperate workspace is created for stages using docker vs not using docker.
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
                    echo "npm dependencies such as node_modules folder installed using npm ci command and then we ran the npm build"
                '''
             }
        }
        */
        stage('Test'){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                # Hash is the format to comment a line in the shell block. commented line in shell block does not turn green. But prepended hash does not execute that command.
                echo 'Test stage'
                #test -f build/index.html
                npm test
                '''
                // to run this npm test in TEST stage, we need node modules also called node dependencies to run it. Without installing node js, the npm test command will throw an error. Thus we again called docker agent in the TEST stage so that it installs node image that contains node modules. Even if we had already done it in the BUILD stage, once the stage is done the docker container is destroyed and thus we need to call the docker agent to install node js again.
            }
        }
        stage('E2E Test'){
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.48.1-noble'
                    reuseNode true
                }
            }
            steps{
                sh '''
                echo 'E2E stage steps'
                //npm install -g serve
                // installs serve tool that would help us install a simple http webserver inorder to test the E2E on some running application. In our case webserver.
                //serve -s build
                // command to start a webserver
               // npx playwright test
                // This would start the test
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
