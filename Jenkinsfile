pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = 'b5e53243-8a3a-4607-9673-94f376c15549'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = '1.2.3'

    }

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

        stage('Tests'){
            parallel{
                stage('Unit Test'){
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
                    post{
                        always{
                            junit 'junit-test-results/junit.xml'
                        }
                    }
                }
                stage('Local Setup E2E Test'){
                    agent{
                        docker{
                            //image 'mcr.microsoft.com/playwright:v1.48.1-noble'
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                        echo 'E2E stage steps'
                        #npm install -g serve - this command installs serve tool GLOBALLY. serve help us install a simple http webserver inorder to test the E2E on some running application. In our case webserver.
                        npm install serve
                        #installs serve tool LOCALLY.
                        #serve -s build - this command starts webserver globally
                        #node_modules/.bin/serve -s build - using serve tool's relative path we are starting the webserver locally instead of globally
                        node_modules/.bin/serve -s build &
                        # & in the end of the command runs it in the background. 
                        sleep 15
                        npx playwright test --reporter=html
                        #This would start the test
                        '''
                    }
                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright local HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy Staging Setup'){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }                  
            steps{
                sh '''
                echo 'Staging Deployment stage'
                npm install netlify-cli node-jq
                node_modules/.bin/netlify --version
                echo "Deploying to Staging. Site Id: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json
                '''
                script{
                    env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
                }
            }
        }

        stage('Staging E2E Test'){
            agent{
                 docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
                    
            environment{
                CI_ENVIRONMENT_URL = "$env.STAGING_URL"
                //single quotes pass the values as string and double quotes pass the value in the variable
            }

            steps{
                sh '''
                echo 'Staging E2E Test'
                npx playwright test --reporter=html
                '''
            }
            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Staging setup HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        /*
        stage('Manual Approval before prod deployment'){             
            steps{
               timeout(time: 1, unit: 'MINUTES'){
                    input message: 'Do you want to proceed for prod deployment?', ok: 'Yes, I want to proceed for prod deployment!'
                }
            }
        }
        */
        stage('Deploy Prod Setup'){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }                  
            steps{
                sh '''
                echo 'Prod Deployment stage'
                npm install netlify-cli
                node_modules/.bin/netlify --version
                echo "Deploying to production. Site Id: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
        stage('Prod E2E Test'){
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    
                    environment{
                        CI_ENVIRONMENT_URL = 'https://roaring-panda-5c6638.netlify.app'
                    }

                    steps{
                        sh '''
                        echo 'Prod setup E2E Test Stage'
                        npx playwright test --reporter=html
                        '''
                    }
                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod setup HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }      
    }
}
