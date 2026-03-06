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
                sh 'oc delete secret entra-id-1-tls --ignore-not-found'
                sh 'oc create secret generic entra-id-1-tls \
                 --from-file=tls.crt=certificates/entra_id_1.crt'
                sh 'oc delete secret entra-id-2-tls --ignore-not-found'
                sh 'oc create secret generic entra-id-2-tls \
                 --from-file=tls.crt=certificates/entra_id_2.crt'
                
                // create secret for Entra ID Web Security
                sh 'oc delete secret entra-id-web-auth-secret --ignore-not-found'
                sh 'oc create secret generic entra-id-web-auth-secret \
                 --from-file=openIdParameters.properties=security/openIdParameters.properties \
                 --from-file=openIdWebSecurity.xml=security/openIdWebSecurity.xml \
                 --from-file=webSecurity.xml=security/webSecurity.xml'
            }
        }

        stage('Deploy ODM to OCP') {
            steps {
                echo 'Deploy ODM to OCP'
                
                sh 'helm uninstall ibm-odm-prod --ignore-not-found'
                sh 'helm install ibm-odm-prod ibm-helm/ibm-odm-prod \
                 --version 24.1.20 \
                 -f charts/ibm-odm-prod/values.yaml'
            }
        }

        stage('Finalization') {
            steps {
                echo 'Finalization'
            }
        }
    }
}
