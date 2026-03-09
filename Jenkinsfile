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
                sh 'oc wait --for=condition=Ready \
                 pod/$(oc get pod -l run=custom-config-app -o jsonpath="{.items[0].metadata.name}") --timeout=300s'

                // copy custom configuration files for DSC and DSR to custom_config folder
                sh '''CUSTOM_CONFIG_APP_POD_NAME=\
                 $(oc get pods -o jsonpath='{.items[0].metadata.name}' --selector=run=custom-config-app)'''
                echo "TESTING"
                sh 'oc cp ./custom_config/application_dsc_custom.xml x:/custom_config/application_dsr_custom.xml'
                //sh '''oc cp ./custom_config/application_dsc_custom.xml \
                // $(oc get pods -o jsonpath='{.items[0].metadata.name}' --selector=run=custom-config-app):/custom_config'''
                //sh 'oc cp ./custom_config/application_dsr_custom.xml \
                // $CUSTOM_CONFIG_APP_POD_NAME:/custom_config/application_dsr_custom.xml'
                //sh 'oc cp ./custom_config/web_dsc_custom.xml \
                // $CUSTOM_CONFIG_APP_POD_NAME:/custom_config/web_dsc_custom.xml'
                //sh 'oc cp ./custom_config/web_dsr_custom.xml \
                // $CUSTOM_CONFIG_APP_POD_NAME:/custom_config/web_dsr_custom.xml'
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
