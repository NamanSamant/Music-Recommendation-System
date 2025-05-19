pipeline {
    agent any

    parameters {
        booleanParam(name: 'RUN_K8S', defaultValue: false, description: 'Deploy to Kubernetes?')
        booleanParam(name: 'RUN_COMPOSE', defaultValue: false, description: 'Run Docker Compose?')
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

                        # Write vault password to a temp file securely
                        VAULT_PASS_FILE=$(mktemp)
                        echo "$VAULT_PASS" > "$VAULT_PASS_FILE"
                        chmod 600 "$VAULT_PASS_FILE"

                        # Run ansible playbook using the vault password file
                        ansible-playbook -i ansible/inventory.ini ansible/playbook.yml --private-key=$SSH_KEY --vault-password-file="$VAULT_PASS_FILE"

                        # Remove the temp vault password file
                        rm -f "$VAULT_PASS_FILE"
                    '''

                    script {
                        if (params.RUN_COMPOSE) {
                            echo "RUN_COMPOSE is true, running docker-compose up"
                            sh '''
                                echo "Starting Docker Compose stack..."
                                docker-compose -f docker-compose.yml up -d
                            '''
                        } else {
                            echo "RUN_COMPOSE is false, skipping docker-compose"
                        }
                    }
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
                        # Skipping configmaps.yml as per request
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
