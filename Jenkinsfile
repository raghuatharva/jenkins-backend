pipeline {
    agent {
        label 'AGENT 1'
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
        account_id = '' // Mention this 
        project = 'expense'
        environment = 'dev'
        component = 'backend'
    }
    stages{
        stage('read the version') {
            steps{
                script {
                    withCredentials([usernamePassword(credentialsId: 'git-credentials', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                        sh 'git clone https://github.com/raghuatharva/jenkins-backend.git'
                        appVersion = sh(script: 'git describe --tags --abbrev=0', returnStdout: true).trim()
                        echo " the latest version is ${appVersion}"
                        
                    }
                }
            }
        }
    }
}