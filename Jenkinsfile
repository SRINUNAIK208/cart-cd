pipeline {
    agent {
        label 'AGENT-1'
    }
    options {
        timeout(time:30, unit: 'MINUTES')
        disableConcurrentBuilds();
    }
    environment {
        appVersion = ''
        region = 'us-east-1'
        Account_ID = '388343452532'
        project = "roboshop"
        component = "cart"
    }
    parameters{
        string(name: 'appVersion', description: 'Image version of application')
        choice(name: 'deploy_to', choices:['dev','qa','prod'], description: 'pick the environment')
    }
    stages {
        stage('Deploy'){
            steps{
                script {
                    withAWS(credentials: 'aws-cred', region: 'us-east-1'){
                    sh """
                      aws eks update-kubeconfig --region ${region} --name ${project}
                      kubectl get nodes
                      sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                      helm upgrade --install ${component} -f values-${params.deploy_to}.yaml -n ${project} .
                    """
                }
                    
                }
            }
        }
        stage('check the deployment status'){
            steps{
                script {
                    withAWS(credentials: 'aws cred', region: 'us-east-1'){
                        def deployStatus = sh(returnStdout: true, script: "kubectl rollout status deployment/${component} --timeout=30s -n roboshop || echo FAILED").trim()
                        if(deployStatus.contains('successfully rolled out'))
                        {
                            echo "deployment is success"
                        }
                        else {
                            sh """
                              helm rollback ${component} -n ${project}
                              sleep 10
                            """
                            
                            def RollbackStatus = sh(script: "kubectl rollout status deployment/${component} --timeout=30s -n roboshop || echo FAILED",returnStdout: true).trim()
                            if(RollbackStatus.contains('successfully rolled out'))
                            {
                                echo "deployment is failed and rollback success"
                            }
                            else {
                                error "deployment failed and rollback also failed "
                            }
                        }
                        


                    }
                }
            }
        }
    }
}