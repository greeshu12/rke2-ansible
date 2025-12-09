pipeline {
    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        ANSIBLE_CONFIG = "${WORKSPACE}/ansible.cfg"

        SSH_USER = "vboxuser"
        MASTER_IP = "10.91.9.235"
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
                ansiblePlaybook(
                    playbook: 'uninstall-rke2.yml',
                    inventory: 'inventory.ini',
                    credentialsId: 'ansible-ssh-key',
                    become: true,
                    becomeUser: 'root',
                    sshUser: 'vboxuser'
                )
            }
        }

        stage('Install RKE2 Server + Agent') {
            steps {
                ansiblePlaybook(
                    playbook: 'install-rke2.yml',
                    inventory: 'inventory.ini',
                    credentialsId: 'ansible-ssh-key',
                    become: true,
                    becomeUser: 'root',
                    sshUser: 'vboxuser'
                )
            }
        }

        stage('Wait For Nodes Ready') {
            steps {
                sh '''
                    echo "⏳ Waiting for Kubernetes nodes to be Ready..."

                    MAX_RETRIES=18   # 3 minutes
                    RETRY=0

                    while [ $RETRY -lt $MAX_RETRIES ]; do

                        STATUS=$(ssh -o StrictHostKeyChecking=no ${SSH_USER}@${MASTER_IP} \
                            "KUBECONFIG=/home/${SSH_USER}/.kube/config /var/lib/rancher/rke2/bin/kubectl get nodes --no-headers" \
                            | awk '{print $2}' | grep -cv Ready)

                        if [ "$STATUS" -eq 0 ]; then
                            echo " "
                            echo "🎉🎉🎉 ALL NODES ARE READY! 🎉🎉🎉"
                            echo "--------------------------------------------------------"
                            echo "📌 FINAL CLUSTER STATE:"
                            ssh -o StrictHostKeyChecking=no ${SSH_USER}@${MASTER_IP} \
                                "KUBECONFIG=/home/${SSH_USER}/.kube/config /var/lib/rancher/rke2/bin/kubectl get nodes -o wide"
                            echo "--------------------------------------------------------"
                            exit 0
                        fi

                        echo "⏳ Nodes not ready yet... ($RETRY/18). Retrying in 10 seconds..."
                        sleep 10
                        RETRY=$((RETRY + 1))
                    done

                    echo "❌ ERROR: Nodes did NOT become Ready in time!"
                    exit 1
                '''
            }
        }
    }
}
