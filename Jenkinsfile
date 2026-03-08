pipeline {
    agent any
    
    parameters {
        string(name: 'PLAYBOOK_NAME', defaultValue: 'playbook.yml', description: 'Enter the .yml filename you want to test')
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
                    file(credentialsId: 'sshkey', variable: 'SSH_KEY')
                ]) {
                    sh """
                    ansible-playbook -i ansible.ini ${params.PLAYBOOK_NAME} \
                    --private-key ${SSH_KEY} \
                    -u ec2-user
                    """
                }
            }
        }
    }
}
