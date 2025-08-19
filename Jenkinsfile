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
        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
    }
    stages {
        stage('Deploy') {
            steps {
                script{
                    withAWS(credentials: 'aws-credds', region: 'us-east-1') {
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