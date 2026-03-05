pipeline {
    agent any

    stages {
        stage('Initialization') {
            steps {
                echo 'Initialization'
                sh '''
                  #!/bin/bash
                  oc projects
                '''
            }
        }
    }
}
