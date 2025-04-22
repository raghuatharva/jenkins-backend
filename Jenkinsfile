
// In actual organistation pipelines:

// ✅ npm install is done outside Docker in CI to:

// Run tests
// Lint code
// npm install downloads all required node modules listed in package.json
//      Without them:
//           ❌ Unit tests fail

// Do SonarQube scan

// Then again npm install (or npm ci) is done inside Docker (via Dockerfile) to:
// Bundle app correctly for prod
// Reduce image size with optimized builds




pipeline {
    agent {
        label 'AGENT-1'
    }
     options{
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        //retry(1)
    }
    environment {
        DEBUG = 'true'
        region= 'us-east-1'
        account_id = '180294178330'
        project = 'expense'
        environment = 'dev'
        component = 'backend'
        APP_VERSION = ''  // this will become global, we can use across pipeline
    }
    stages{
        stage('read the version') {
            steps {
                script {
                    sh 'git clone https://github.com/raghuatharva/jenkins-backend.git'
                    APP_VERSION = sh(script: 'cd jenkins-backend && git describe --tags --abbrev=0', returnStdout: true).trim()
                    echo "The latest version is ${env.APP_VERSION}"
                }
                // this is the only and best way to get the tag version. clone it , cd into cloned repo and get the latest tag version. 
            }
        }
        // stage('Read the version') {
        //     steps {
        //         script{
        //             def packageJson = readJSON file: 'package.json'
        //             appVersion = packageJson.version
        //             echo "App version: ${appVersion}"
        //         }
        //     }
        // }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Sonarqube analysis') {   
            // ಇದು ಯಾವ ಸೋನಾರ್ environment ಅಂತ ಹೇಳುತ್ತೆ.  it may be sonar 6.0 versions , 7.0 or any version
            environment{
                SCANNER_HOME = tool 'sonar-anna' 
                // iam using sonar-scanner 7.9.1 under the name "sonar-anna"(manage jenkins > tools > sonar-scanner)
            }
            steps{
                withSonarQubeEnv('sonar-ellidiyappa'){ 
                   sh '${SCANNER_HOME}/bin/sonar-scanner'
                }
                // sonar-ellidiyappa is the name of the sonar server which is configured in manage jenkins > system > sonar servers
                // generic scanner, it automatically understands any scripting language(if the source code is java ,python etc.) and provide scan results
                // this shell command sh '${SCANNER_HOME}/bin/sonar-scanner' --> the sonar scanner we installed  reads the sonar-project.properties file and sends the data to the sonar server with the help of pluggin. what will sonar server do is, it will scan the code and provide the results
            }
        }

        stage('Sonar Quality gate'){
            // above stage will run the sonar scanner and send the data to the sonar server. we should create a quality gate in the sonar server. quality gate is a set of conditions that the code must meet. if the code meets the conditions, then it is considered "good" or "acceptable". if the code does not meet the conditions, then it is considered "bad" or "unacceptable". if the code is unacceptable, then the pipeline will be aborted. 
            // this stage will check whether the code is passed or failed
            steps{
                script {
                    timeout(time: 10, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
            // this will check the quality gate status. if it is passed, then it will continue the pipeline. if it is failed, then it will abort the pipeline
            //  some of the conditions are:
            // 1. code coverage: the percentage of code that is covered by tests; the value should be greater than 80%
            // 2. code smells: the number of code smells in the code; the value should be less than 10
            // 3. bugs: the number of bugs in the code; the value should be less than 5
            // 4. vulnerabilities: the number of vulnerabilities in the code; the value should be less than 5
            // 5. duplications: the percentage of code that is duplicated; the value should be less than 3%
            // 6. security hotspots: the number of security hotspots in the code; the value should be less than 5
            // 7. maintainability rating: the rating of the code; the value should be A or B
            // 8. reliability rating: the rating of the code; the value should be A or B

            //                              -------MOST IMPORTANT -------

            // JUST WRITING THIS BLOCK IS NOT ENOUGH ; TO STOP THE PIPELINE , SONARQUBE SERVER SHOULD BE CONFIGURED WITH THE WEBHOOK; TO DO THAT GO TO SONAR SERVER --> ACCOUNT --> ADMIN --> CONFIGURATION --> WEBHOOKS and add this url ; < jenkins.rohanandlife.site:9000/sonarqube-webhook/ 

            // this will check the quality gate status. if it is passed, then it will continue the pipeline. if it is failed, then it will abort the pipeline
        }

        stage('building docker image'){
            steps{
                withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                sh """
                aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.us-east-1.amazonaws.com
                docker build -t ${account_id}.dkr.ecr.${region}.amazonaws.com/${project}/${environment}/${component}:${APP_VERSION} .
                docker push ${account_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${environment}/${component}:${APP_VERSION}
                """
                }
            }
        }
        stage('deployment'){
            steps{
                withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                    sh """
                    aws eks update-kubeconfig --region ${region} --name ${project}-${environment}
                    cd helm
                    sed -i s/IMAGE_VERSION/${APP_VERSION}/g values.yaml
                    helm upgrade --install ${component} -n ${project} .
                    """
                }
            }
        }
    }
        
    post{
        always{
            echo "this will run always"
            deleteDir()
        }
        success{
            echo "BACKEND IS DEPLOYED SUCCESSFULLY"
        }
        failure{
            echo " PIPELINE IS FAILED"
        }
    }
}