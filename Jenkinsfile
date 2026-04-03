pipeline {
    agent any

    environment {
        HETZNER_HOST = '89.167.27.46'
        HETZNER_USER = 'root'
        REPO_URL     = 'https://github.com/harshana96/helm-k8s-practice.git'
    }

    stages {

        stage('Clone Repo on Hetzner') {
            steps {
                sshagent(['hetzner-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${HETZNER_USER}@${HETZNER_HOST} '
                            if [ -d ~/helm-k8s-practice ]; then
                                cd ~/helm-k8s-practice && git pull
                            else
                                git clone ${REPO_URL} ~/helm-k8s-practice
                            fi
                        '
                    """
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                sshagent(['hetzner-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${HETZNER_USER}@${HETZNER_HOST} '
                            cd ~/helm-k8s-practice
                            export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
                            kubectl create namespace dev --dry-run=client -o yaml | kubectl apply -f -
                            helm upgrade --install nginx-dev ./helm \
                                -f ./helm/values-dev.yaml \
                                --namespace dev \
                                --atomic \
                                --timeout 60s
                        '
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sshagent(['hetzner-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${HETZNER_USER}@${HETZNER_HOST} '
                            export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
                            kubectl get pods -n dev
                            kubectl get svc -n dev
                            kubectl get ingress -n dev
                        '
                    """
                }
            }
        }

    }

    post {
        success {
            echo '✅ Deployment to Dev successful!'
        }
        failure {
            echo '❌ Deployment failed!'
        }
    }
}