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
                sh 'oc delete secret entra-id-3-tls --ignore-not-found'
                sh 'oc create secret generic entra-id-3-tls \
                 --from-file=tls.crt=certificates/entra_id_3.crt'
                
                sh 'oc delete secret microsoft-ecc-root-ca-tls --ignore-not-found'
                sh 'oc create secret generic microsoft-ecc-root-ca-tls \
                 --from-file=tls.crt=certificates/microsoft-ecc-root-ca.crt'
                sh 'oc delete secret microsoft-rsa-root-ca-tls --ignore-not-found'
                sh 'oc create secret generic microsoft-rsa-root-ca-tls \
                 --from-file=tls.crt=certificates/microsoft-rsa-root-ca.crt' 
                
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
                // update deployments to add custom config mounts
                sh 'oc patch deployment/ibm-odm-prod-odm-decisionserverconsole \
                 --patch "$(cat patches/deployment/ibm-odm-prod-odm-decisionserverconsole.yaml)" \
                 --type=strategic'
                
                //sh 'oc set volume deployment/ibm-odm-prod-odm-decisionserverconsole \
                // --add --name=custom-volume \
                // --type=persistentVolumeClaim \
                // --claim-name=custom-pvc \
                // --mount-path=/custom_config'
                
                //sh '''oc patch deployment/ibm-odm-prod-odm-decisionserverconsole \
                // -p '{"spec": {"template": {"spec": {"initContainers": [{"name": "init-folder-readonlyfs", "volumeMounts": [{"name": "custom-volume", "mountPath": "/custom_config" }]}]}}}}' \
                // --type=strategic'''
            }
        }
    }
}
