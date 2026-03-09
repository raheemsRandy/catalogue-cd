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
                            aws eks update-kubeconfig --region $REGION --name "$PROJECT-${params.deploy_to}"
                            kubectl get nodes
                            kubectl apply -f 01-namespace.yaml
                            sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                            helm upgrade --install $COMPONENT -f values-${params.deploy_to}.yaml -n $PROJECT .
                        """
                    }
                }
            }
        }
        stage ('Check Status'){
            steps {
                script{
                 withAWS(credentials: 'aws-creds', region: REGION) {
                    def deploymentStatus = sh(returnStdout: true,script: "kubectl rollout status deployment/catalogue --timeout=60s -n $PROJECT || echo  FAILED").trim()
                    if (deploymentStatus.contains("successfully rolled out")) {
                       echo "Deployment is success"
                    } else {
                       sh """
                            helm rollback $COMPONENT -n $PROJECT
                            sleep 20
                       """
                       def rollbackStatus = sh(returnStdout: true,script: "kubectl rollout status deployment/catalogue --timeout=60s -n $PROJECT || echo  FAILED").trim()
                        if (rollbackStatus.contains("successfully rolled out")) {
                            error "Deployment is Failure,Rollback Success"
                        }
                        else{
                            error "Deployment is Failure,Rollback Failure. Application not running"
                        }  
                    }
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