pipeline {
    agent any
    
    parameters {
        string(name: 'PLAYBOOK_NAME', defaultValue: 'playbook.yml', description: 'Enter the .yml filename you want to test')
        string(name: 'TAGS', defaultValue: '', description: 'Enter tags to run specific tasks (leave empty for all tasks)')
    }

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                withCredentials([
                    file(credentialsId: 'sshkey', variable: 'SSH_KEY'),
                    string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY'),
                    string(credentialsId: 'aws-default-region', variable: 'AWS_DEFAULT_REGION')
                ]) {
                    sh """
                    export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                    export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                    export AWS_DEFAULT_REGION=${AWS_DEFAULT_REGION}
                    ansible-playbook -i ansible.ini ${params.PLAYBOOK_NAME} \
                    --private-key ${SSH_KEY} \
                    -u ec2-user \
                    --vault-password-file vault.yml \
                    ${params.TAGS ? "--tags ${params.TAGS}" : ""}
                    """
                }
            }
        }
    }
}
