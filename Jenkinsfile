pipeline {
    agent any

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
                    sh '''
                    ansible-playbook -i ansible.ini playbook.yml \
                    --private-key $SSH_KEY \
                    -u centos
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Ansible playbook executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}