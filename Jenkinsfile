pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'v1.0.4', description: 'Docker image tag to deploy')
    }

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }

    stages {
        stage('Get Code') {
            steps {
                checkout scm
            }
        }

        stage('Deploy Container') {
            steps {
                withCredentials([
                    string(credentialsId: 'ansible-vault-pass', variable: 'VAULT_PASS')
                ]) {
                    sh '''
                        echo "$VAULT_PASS" > vault_pass.txt
                        ansible-playbook \
                          -i ansible/inventory/hosts.ini \
                          ansible/playbooks/deploy.yml \
                          --extra-vars "image_tag=${IMAGE_TAG}" \
                          --vault-password-file vault_pass.txt
                    '''
                }
            }
        }

        stage('Verify Application') {
            steps {
                withCredentials([
                    string(credentialsId: 'ansible-vault-pass', variable: 'VAULT_PASS')
                ]) {
                    sh '''
                        echo "$VAULT_PASS" > vault_pass.txt
                        ansible-playbook \
                          -i ansible/inventory/hosts.ini \
                          ansible/playbooks/verify.yml \
                          --vault-password-file vault_pass.txt
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'rm -f vault_pass.txt'
        }
    }
}