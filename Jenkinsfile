pipeline {
    agent any
    parameters {
        choice(name: 'TARGET_ENV', choices: ['blue', 'green'], description: 'Select environment to deploy and switch traffic to')
    }
    environment {
        KUBE = "--kubeconfig=/var/jenkins_home/.kube/config --insecure-skip-tls-verify"
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/divakar-2005/k8s-blue-green-demo.git'
            }
        }
        stage('Build & Load Image') {
            steps {
                sh "docker build -t bg-demo:${params.TARGET_ENV} ."
                sh "docker save bg-demo:${params.TARGET_ENV} -o /tmp/bg.tar && docker cp /tmp/bg.tar desktop-control-plane:/root/bg.tar && docker exec desktop-control-plane ctr --namespace=k8s.io images import /root/bg.tar"
            }
        }
        stage('Deploy Target Environment') {
            steps {
                sh "kubectl ${KUBE} apply -f ${params.TARGET_ENV}-deployment.yaml"
                sh "kubectl ${KUBE} apply -f service.yaml"
                sh "kubectl ${KUBE} rollout restart deployment/web-${params.TARGET_ENV}"
                sh "kubectl ${KUBE} rollout status deployment/web-${params.TARGET_ENV}"
            }
        }
        stage('Switch Live Traffic') {
            steps {
                sh """kubectl ${KUBE} patch service bg-demo-service -p '{"spec":{"selector":{"app":"bg-demo","version":"${params.TARGET_ENV}"}}}'"""
            }
        }
    }
}
