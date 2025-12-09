pipeline {
    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        ANSIBLE_CONFIG = "${WORKSPACE}/ansible.cfg"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/greeshu12/rke2-ansible.git'
            }
        }

        stage('Uninstall Existing RKE2') {
            steps {
                ansiblePlaybook credentialsId: 'ansible-ssh-key',
                                  playbook: 'uninstall-rke2.yml'
            }
        }

        stage('Install RKE2 Server + Agent') {
            steps {
                ansiblePlaybook credentialsId: 'ansible-ssh-key',
                                  playbook: 'install-rke2.yml'
            }
        }

        stage('Wait For Nodes Ready') {
            steps {
                sh '''
                echo "⏳ Waiting for nodes to be Ready..."

                MAX_RETRIES=18
                RETRY=0

                while [ $RETRY -lt $MAX_RETRIES ]; do
                    STATUS=$(ssh -o StrictHostKeyChecking=no vboxuser@10.91.9.235 \
                        "sudo /var/lib/rancher/rke2/bin/kubectl get nodes --no-headers" \
                        | awk '{print $2}' | grep -cv Ready)

                    if [ "$STATUS" -eq 0 ]; then
                        echo "✅ All nodes are Ready!"
                        ssh -o StrictHostKeyChecking=no vboxuser@10.91.9.235 \
                            "sudo /var/lib/rancher/rke2/bin/kubectl get nodes -o wide"
                        exit 0
                    fi

                    echo "⏳ Not ready yet... retrying..."
                    sleep 10
                    RETRY=$((RETRY+1))
                done

                echo "❌ ERROR: Nodes did not become Ready in time!"
                exit 1
                '''
            }
        }
    }
}
