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
    }
    stages{
        stage('read the version') {
            steps {
                script {
                    sh 'git clone https://github.com/raghuatharva/jenkins-backend.git'
                    env.APP_VERSION = sh(script: 'git describe --tags --abbrev=0', returnStdout: true).trim()
                    echo "The latest version is ${env.APP_VERSION}"
                }
            }
        }

        stage('building docker image'){
            steps{
                withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                sh """
                aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.us-east-1.amazonaws.com
                docker build -t ${account_id}.dkr.ecr.${region}.amazonaws.com/${project}/${environment}/${component}:${env.APP_VERSION} .
                docker push ${account_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${environment}/${component}:${env.APP_VERSION}
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
                    sed -i s/IMAGE_VERSION/${env.APP_VERSION}/g values.yaml
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