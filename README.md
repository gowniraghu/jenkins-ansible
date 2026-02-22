Jenkins + Ansible Setup

This project shows how to run an Ansible playbook from Jenkins pipeline using SSH credentials stored securely in Jenkins.

Prerequisites

Jenkins installed

Ansible installed on Jenkins server

Remote Linux server (EC2 or VM)

SSH access to remote server

Port 22 open in security group / firewall

Verify Ansible:

ansible --version
Jenkins Credential Setup

Go to Manage Jenkins → Credentials

Click Add Credentials

Select:

Kind: Secret file

ID: sshkey

Upload your .pem private key file

Save

This securely stores your SSH key inside Jenkins.

Inventory File (ansible.ini)
[web]
172.31.33.64 ansible_user=ec2-user

Replace IP with your server IP

Make sure the correct SSH user is used

Sample Playbook (playbook.yml)
- name: Install and start httpd
  hosts: web
  become: yes

  tasks:
    - name: Install httpd
      package:
        name: httpd
        state: present

    - name: Start httpd
      service:
        name: httpd
        state: started
        enabled: yes
Jenkinsfile
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
                    --private-key $SSH_KEY
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
How It Works

Jenkins pulls code from GitHub.

Jenkins loads SSH key from credentials.

Ansible connects to remote server.

Playbook installs and starts httpd service.

Common Issues
Permission denied (publickey)

Wrong SSH user

Wrong key

Key not present in authorized_keys

Test manually:

ssh -i key.pem ec2-user@server-ip
Project Structure
.
├── Jenkinsfile
├── ansible.ini
├── playbook.yml
└── README.md
Result

After successful build:

Ansible runs from Jenkins

httpd is installed

Service is started

Pipeline shows SUCCESS

This setup keeps SSH keys secure and avoids hardcoding secrets in the repository.