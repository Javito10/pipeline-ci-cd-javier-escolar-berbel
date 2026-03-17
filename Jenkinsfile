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
		    rm -rf .venv
                    python3 -m venv .venv
                    . .venv/bin/activate
                    python3 -m pip install --upgrade pip
                    python3 -m pip install flask pytest
                    python3 -mpytest
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
		sh 'kubectl --kubeconfig=/var/lib/jenkins/.kube/config rollout restart deployment python-app'
            }
        }
    }
}
