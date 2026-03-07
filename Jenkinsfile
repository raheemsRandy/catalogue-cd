pipeline {

    agent {
        label 'Agent1'
    }

    environment {
        appVersion = ''
        REGION = 'us-east-1'
        ACC_ID ='989088456804'
        PROJECT = 'roboshop'
        COMPONENT = 'catalogue'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    parameters {
        string ( name: 'appVersion', description: 'image version of the application')
        choice (name: 'deploy_to', choices:['dev', 'qa', 'prod'], description: 'Pick the Environment')
    }
    stages {

        stage('deploy') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                        sh """
                            aws eks update kubeconfig --region $REGION --name "$PROJECT-${params.deploy_to}"
                           
                            kubectl get nodes
                            sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                            helm upgrade --install $COMPONENT -f values-${params.deploy_to}.yaml .
                        """
                    }
                }
            }
        }

    }

    post {
        always {
            echo 'Cleaning workspace'
            deleteDir()
        }

        success {
            echo 'Pipeline Success'
        }

        failure {
            echo 'Pipeline Failed'
        }
    }
}