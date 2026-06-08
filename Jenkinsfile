pipeline {
    agent { label 'AGENT-1' }
    environment { 
        PROJECT = 'expense'
        COMPONENT = 'frontend'
        APP_VERSION = ''   // use uppercase consistently
        ACC_ID = '894650614410'
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
                script {
                    def packageJson = readJSON file: 'package.json'
                    env.APP_VERSION = packageJson.version   // assign to env
                    echo "Version is: ${env.APP_VERSION}"
                }
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                        sh """
                        aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${env.ACC_ID}.dkr.ecr.us-east-1.amazonaws.com

                        docker build \
                          --platform linux/amd64 \
                          --provenance=false \
                          --sbom=false \
                          -t ${env.ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${env.PROJECT}/${env.COMPONENT}:${env.APP_VERSION} .

                        docker push ${env.ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${env.PROJECT}/${env.COMPONENT}:${env.APP_VERSION}
                        """
                    }
                }
            }
        }

        stage('Trigger Deploy') {
            when { expression { params.deploy } }
            steps {
                build job: 'frontend-cd',
                      parameters: [string(name: 'version', value: "${env.APP_VERSION}")],
                      wait: true
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
