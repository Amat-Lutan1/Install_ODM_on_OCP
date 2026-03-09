pipeline {
    agent any

    stages {
        stage('Initialization') {
            steps {
                echo 'Initialization'

                // switch project namespace for ODM installation
                sh 'oc project odm-sandbox'

                // add helm repository for IBM Helm
                sh 'helm repo add ibm-helm https://raw.githubusercontent.com/IBM/charts/master/repo/ibm-helm'
                sh 'helm repo update'
                
                // create secret for Entra ID TLS certificates
                sh 'oc delete secret ms-secret --ignore-not-found'
                sh 'oc create secret generic ms-secret \
                 --from-file=tls.crt=certificates/microsoft.crt'               
                sh 'oc delete secret digicert-secret --ignore-not-found'
                sh 'oc create secret generic digicert-secret \
                 --from-file=tls.crt=certificates/DigiCertGlobalRootG2.crt.pem'
                
                // create secret for Entra ID Web Security
                sh 'oc delete secret web-auth-secret --ignore-not-found'
                sh 'oc create secret generic web-auth-secret \
                 --from-file=openIdParameters.properties=security/openIdParameters.properties \
                 --from-file=openIdWebSecurity.xml=security/openIdWebSecurity.xml \
                 --from-file=webSecurity.xml=security/webSecurity.xml'

                // create custom config app for mounting the custom config pvc
                sh 'oc delete deployment custom-config-app'
                sh 'oc apply -f deployment/custom-config-app.yaml'
                sh 'oc wait --for=condition=Ready pod -l run=custom-config-app'
            }
        }

        stage('Deploy ODM to OCP') {
            steps {
                echo 'Deploy ODM to OCP'
                // install ODM using helm chart
                sh 'helm uninstall ibm-odm-prod --ignore-not-found'
                sh 'helm install ibm-odm-prod ibm-helm/ibm-odm-prod \
                 --version 24.1.20 \
                 -f charts/ibm-odm-prod/values.yaml \
                 --wait'
            }
        }

        stage('Finalization') {
            steps {
                echo 'Finalization'
                // update decision server console deployment to add custom config mount
                sh 'oc patch deployment/ibm-odm-prod-odm-decisionserverconsole \
                 --patch-file=patches/deployment/ibm-odm-prod-odm-decisionserverconsole.yaml \
                 --type=strategic'
                
                sh 'oc scale deployment/ibm-odm-prod-odm-decisionserverconsole --replicas=1'

                // update decision server runtime deployment to add custom config mount
                sh 'oc patch deployment/ibm-odm-prod-odm-decisionserverruntime \
                 --patch-file=patches/deployment/ibm-odm-prod-odm-decisionserverruntime.yaml \
                 --type=strategic'
                
                sh 'oc scale deployment/ibm-odm-prod-odm-decisionserverruntime --replicas=1'
            }
        }
    }
}
