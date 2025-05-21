pipeline {
    agent { label 'jenkins-agent' }
    environment { 
        PROJECT = 'cluster'
        COMPONENT = 'backend'
        ACC_ID = '010526266250'
    }
    options {
        disableConcurrentBuilds()
        timeout(time: 30, unit: 'MINUTES')
    }
    parameters {
        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
    }
    stages {
        stage('Read Version') {
            steps {
                dir('Backend') {
                    script {
                        def packageJson = readJSON file: 'package.json'
                        // Assign to a Groovy variable
                        appVersion = packageJson.version
                        echo "Version is: ${appVersion}"
                    }
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                dir('Backend') {
                    sh 'npm install'
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    withAWS(region: 'us-east-1', credentials: 'aws') {
                        sh """
                        aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                        docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} Backend/
                        docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                        """
                    }
                }
            }
        }
        stage('Trigger Deploy'){
            when { 
                expression { params.deploy }
            }
            steps {
                build job: 'helm-k8', parameters: [string(name: 'version', value: "${appVersion}")], wait: true
            }
        }
    }
    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()
        }
        failure { 
            echo 'I will run when pipeline is failed'
        }
        success { 
            echo 'I will run when pipeline is success'
        }
    }
}
