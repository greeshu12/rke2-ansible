pipeline {
    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        KUBECONFIG = "/var/lib/jenkins/workspace/rke2-pipeline/kubeconfig"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/greeshu12/rke2-ansible.git', branch: 'main', credentialsId: 'ansible-ssh-key'
            }
        }

        stage('Uninstall Old RKE2') {
            steps {
                sh 'ansible-playbook -i inventory.ini uninstall-rke2.yml || true'
            }
        }

        stage('Install RKE2 Server + Agent') {
            steps {
                sh 'ansible-playbook -i inventory.ini install-rke2.yml'
            }
        }

        stage('Wait for Nodes to Become Ready') {
            steps {
                script {
                    echo "⏳ Waiting for master and worker to become Ready (max 2 minutes)..."

                    retry(12) {      // 12 retries × 10 sec = 2 minutes
                        sleep 10
                        sh '''
                        export KUBECONFIG=/var/lib/jenkins/workspace/rke2-pipeline/kubeconfig
                        READY=$(kubectl get nodes --no-headers | grep -c " Ready")
                        if [ "$READY" -lt 2 ]; then
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
                export KUBECONFIG=/var/lib/jenkins/workspace/rke2-pipeline/kubeconfig
                echo "🎉 FINAL CLUSTER STATUS:"
                kubectl get nodes -o wide
                '''
            }
        }
    }
}
