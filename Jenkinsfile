pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'authservice'
        DOCKER_TAG = 'latest'
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                    withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        // Ensure the image name is lowercase
                        def dockerImageWithRepo = "${DOCKER_USERNAME.toLowerCase()}/${DOCKER_IMAGE.toLowerCase()}:${DOCKER_TAG.toLowerCase()}"
                        sh """
                            docker build -t ${dockerImageWithRepo} . > /dev/null 2>&1
                        """
                    }
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        echo 'Logging in to Docker Hub...'
                        // Secure Docker login, do not echo sensitive variables
                        sh """
                            echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin > /dev/null 2>&1
                        """
                        def dockerImageWithRepo = "${DOCKER_USERNAME.toLowerCase()}/${DOCKER_IMAGE.toLowerCase()}:${DOCKER_TAG.toLowerCase()}"
                        // Pushing Docker image, again suppress output
                        sh "docker push ${dockerImageWithRepo} > /dev/null 2>&1"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'EKS-1', contextName: '', credentialsId: 'k8-token', namespace: 'auth', serverUrl: 'https://7302D1DF066773D16142E09F2D140FC0.sk1.ap-south-2.eks.amazonaws.com']]) {
                    echo 'Deploying application to Kubernetes...'
                    // Apply deployment to Kubernetes, suppress output
                    sh "kubectl apply -f deployment-service.yml > /dev/null 2>&1"
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'EKS-1', contextName: '', credentialsId: 'k8-token', namespace: 'auth', serverUrl: 'https://7302D1DF066773D16142E09F2D140FC0.sk1.ap-south-2.eks.amazonaws.com']]) {
                    echo 'Verifying deployment...'
                    // Verify all resources in the Kubernetes auth namespace, suppress output
                    sh "kubectl get all -n auth > /dev/null 2>&1"
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed!'
        }
        success {
            echo 'Build, deployment, and verification successful!'
        }
        failure {
            echo 'An error occurred during the pipeline execution.'
        }
    }
}
 // this code is worked and eployed into aws cluster