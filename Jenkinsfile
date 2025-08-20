pipeline {
    agent {
        label "AGENT-1"
    }
    environment {
        // COURSE = "JENKINS"
        appVersion = ""
        REGION="us-east-1"
        ACC_ID="010666226306"
        PROJECT="roboshop"
        COMPONENT="catalogue"
    }
    options {
        timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }
    parameters {
        string(name: 'appVersion', description: 'Image version of the application')
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick the Environment')
    }
    stages {
        stage('Deploy') {
            steps {
                script{
                    withAWS(credentials: 'aws-credds', region: 'us-east-1') {
                        sh """
                            aws eks update-kubeconfig --region $REGION --name "$PROJECT-${params.deploy_to}"
                            kubectl get nodes
                            kubectl apply -f 01_namespace.yaml
                            sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                            helm upgrade --install $COMPONENT -f values-${params.deploy_to}.yaml -n $PROJECT .
                        """
                    }
                }
            }
            
        }
         stage('Check Status'){
            steps{
                script{
                    withAWS(credentials: 'aws-credds', region: 'us-east-1') {
                        def deploymentStatus = sh(returnStdout: true, script: "kubectl rollout status deployment/catalogue --timeout=30s -n $PROJECT || echo FAILED").trim()
                        if (deploymentStatus.contains("successfully rolled out")) {
                            echo "Deployment is success"
                        } else {
                            sh """
                                helm rollback $COMPONENT -n $PROJECT
                                sleep 20
                            """
                            def rollbackStatus = sh(returnStdout: true, script: "kubectl rollout status deployment/catalogue --timeout=30s -n $PROJECT || echo FAILED").trim()
                            if (rollbackStatus.contains("successfully rolled out")) {
                                error "Deployment is Failure, Rollback Success"
                            }
                            else{
                                error "Deployment is Failure, Rollback Failure. Application is not running"
                            }
                        }

                    }
                }
            }
        }
    }


      
    }


    post {
        always {
            echo " always hello"
            deleteDir()
        }
        success {
            echo " success hello"
        }
        failure {
            echo " failure hello"
        }
    }
}