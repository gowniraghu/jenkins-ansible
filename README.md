# 🚀 Jenkins + Ansible Setup

This project demonstrates how to run an **Ansible playbook from a
Jenkins pipeline** using SSH credentials stored securely in Jenkins.

------------------------------------------------------------------------

## 📌 Prerequisites

-   Jenkins installed and running
-   Ansible installed on Jenkins server
-   Remote Linux server (EC2 or VM)
-   SSH access to remote server
-   Port 22 open in security group / firewall

### Verify Ansible Installation

    ansible --version

------------------------------------------------------------------------

## 🔐 Jenkins Credential Setup

1.  Go to **Manage Jenkins → Credentials**
2.  Click **Add Credentials**
3.  Select:
    -   **Kind:** Secret file
    -   **ID:** sshkey
4.  Upload your `.pem` private key file
5.  Click **Save**

This securely stores your SSH key inside Jenkins and avoids hardcoding
secrets in the repository.

------------------------------------------------------------------------

## 📁 Inventory File (ansible.ini)

    [web]
    172.31.33.64 ansible_user=ec2-user

-   Replace the IP address with your server IP.
-   Ensure the correct SSH username is used.

------------------------------------------------------------------------

## 📜 Sample Playbook (playbook.yml)

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

------------------------------------------------------------------------

## 🧾 Jenkinsfile

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
                        ansible-playbook -i ansible.ini playbook.yml                         --private-key $SSH_KEY
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

------------------------------------------------------------------------

## ⚙ How It Works

1.  Jenkins pulls code from GitHub.
2.  Jenkins loads the SSH key securely from Credentials.
3.  Ansible connects to the remote server.
4.  The playbook installs and starts the httpd service.
5.  Pipeline completes successfully.

------------------------------------------------------------------------

## 🚨 Common Issues

### Permission denied (publickey)

Possible causes:

-   Wrong SSH username
-   Wrong private key
-   Public key not present in authorized_keys

Test manually:

    ssh -i key.pem ec2-user@server-ip

------------------------------------------------------------------------

## 📂 Project Structure

    .
    ├── Jenkinsfile
    ├── ansible.ini
    ├── playbook.yml
    └── README.md

------------------------------------------------------------------------

## ✅ Result

After a successful Jenkins build:

-   Ansible runs from Jenkins
-   httpd is installed
-   Service is started
-   Pipeline shows SUCCESS

------------------------------------------------------------------------

## 🔒 Security Note

-   Never commit private keys to GitHub.
-   Always use Jenkins Credentials for storing secrets.
-   Keep SSH access limited and secure.
