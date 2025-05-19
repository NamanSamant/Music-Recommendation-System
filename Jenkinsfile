pipeline {
    agent any

    parameters {
        booleanParam(name: 'RUN_K8S', defaultValue: false, description: 'Deploy to Kubernetes?')
    }

    stages {

        stage('Run Ansible Playbook') {
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'SSH_KEY'),
                    string(credentialsId: 'ansible-vault-password', variable: 'VAULT_PASS')
                ]) {
                    sh '''
                #!/bin/bash
                echo "Running Ansible Playbook..."
                export ANSIBLE_HOST_KEY_CHECKING=False
                ansible-playbook -i inventory.ini playbook.yml --private-key=$SSH_KEY --vault-password-file=$VAULT_PASS
                '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                expression { return params.RUN_K8S }
            }
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-secret-file', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                        echo "Deploying to Kubernetes..."
                        export KUBECONFIG=$KUBECONFIG_FILE
                        kubectl apply -f k8s/backend.yml
                        kubectl apply -f k8s/frontend.yml
                        kubectl apply -f k8s/ml_service.yml
                        kubectl apply -f k8s/elasticsearch.yml
                        kubectl apply -f k8s/kibana.yml
                        kubectl apply -f k8s/logstash.yml
                        kubectl apply -f k8s/configmaps.yml
                        kubectl apply -f k8s/pv.yml
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline complete.'
        }
    }
}
