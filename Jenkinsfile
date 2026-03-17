pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "javitoo10/python-app:latest"
    }

    stages {
        stage('clone') {
            steps {
                checkout scm
            }
        }

        stage('test') {
            steps {
                sh '''
                    python3 -m venv .venv
                    . .venv/bin/activate
                    pip install --upgrade pip
                    pip install flask pytest
                    pytest
                '''
            }
        }

        stage('build image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('dockerhub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }

        stage('deploy') {
            steps {
                sh 'kubectl --kubeconfig=/var/lib/jenkins/.kube/config apply -f deployment.yaml'
                sh 'kubectl --kubeconfig=/var/lib/jenkins/.kube/config apply -f service.yaml'
            }
        }
    }
}
