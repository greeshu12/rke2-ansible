pipeline {
    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/greeshu12/rke2-ansible.git'
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                sshagent(['ansible_ssh_key']) {
                    sh '''
                        ansible-playbook -i inventory.ini install-rke2.yml --become
                    '''
                }
            }
        }
    }
}
