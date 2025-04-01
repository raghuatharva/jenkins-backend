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
        appVersion= ''
        region= 'us-east-1'
        account_id = '180294178330'
        project = 'expense'
        environment = 'dev'
        component = 'backend'
    }
    stages{
        stage('read the version') {
            steps{
                sh 'git clone https://github.com/raghuatharva/jenkins-backend.git'
                appVersion = sh(script: 'git describe --tags --abbrev=0', returnStdout: true).trim()
                echo " the latest version is ${appVersion}"
            }
        }
        stage('building docker image'){
            steps{
                withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                sh """
                aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.us-east-1.amazonaws.com
                docker build -t ${account_id}.dkr.ecr.${region}.amazonaws.com/${project}/${environment}/${component}:${appVersion} .
                docker push ${account_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${environment}/${component}:${appVersion}
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
                    sed -i s/IMAGE_VERSION/${appVersion}/g values.yaml
                    helm install --upgrade ${component} -n ${project}
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

