pipeline {
    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        KUBECONFIG = "kubeconfig"
        PATH = "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/bin"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/greeshu12/rke2-ansible.git', 
                    branch: 'main', 
                    credentialsId: 'ansible-ssh-key'
            }
        }

        stage('Uninstall Old RKE2') {
            steps {
                sh 'ansible-playbook -i inventory.ini uninstall-rke2.yml || true'
            }
        }

        stage('Install RKE2 Server + Worker') {
            steps {
                sh 'ansible-playbook -i inventory.ini install-rke2.yml'
            }
        }

        stage('Fetch Kubeconfig from Master') {
            steps {
                sh '''
                ansible master1 -i inventory.ini -m fetch \
                    -a "src=/etc/rancher/rke2/rke2.yaml dest=kubeconfig flat=yes" \
                    -u vboxuser --private-key /var/lib/jenkins/.ssh/id_rsa -b

                sed -i 's/127.0.0.1/10.91.9.235/g' kubeconfig

                # Fix permissions so Jenkins can read it
                sudo chown jenkins:jenkins kubeconfig
                sudo chmod 600 kubeconfig
                '''
            }
        }

        stage('Wait for Nodes Ready') {
            steps {
                script {
                    echo "⏳ Waiting for nodes to become Ready..."

                    retry(12) {
                        sleep 10
                        sh '''
                        export KUBECONFIG=kubeconfig

                        echo "Checking nodes..."
                        kubectl get nodes --no-headers || true

                        READY_COUNT=$(kubectl get nodes --no-headers | grep -c " Ready")

                        if [ "$READY_COUNT" -lt 2 ]; then
                            echo "Nodes not ready yet..."
                            exit 1
                        fi
                        '''
                    }
                }
            }
        }

        stage('Show Final Cluster Status') {
            steps {
                sh '''
                export KUBECONFIG=kubeconfig

                echo "🎉 FINAL CLUSTER STATUS:"
                kubectl get nodes -o wide
                '''
            }
        }
    }
}
