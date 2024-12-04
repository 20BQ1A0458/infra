pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'authservice'
        DOCKER_TAG = 'latest'
    }

    stages {
        stage('Determine Branch') {
            steps {
                script {
                    BRANCH_NAME = env.BRANCH_NAME
                    echo "Running pipeline for branch: ${BRANCH_NAME}" // Logs branch name only (safe)
                }
            }
        }

        stage('Build Docker Image') {
            when {
                anyOf {
                    branch 'feature'
                    branch 'master'
                }
            }
            steps {
                script {
                    echo 'Building Docker image...'
                    withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        def dockerImageWithRepo = "${DOCKER_USERNAME.toLowerCase()}/${DOCKER_IMAGE.toLowerCase()}:${DOCKER_TAG.toLowerCase()}"
                        sh "docker build -t ${dockerImageWithRepo} ."
                    }
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            when {
                branch 'master'
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        echo 'Logging in to Docker Hub...'
                        sh """
                            echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin
                        """
                        def dockerImageWithRepo = "${DOCKER_USERNAME.toLowerCase()}/${DOCKER_IMAGE.toLowerCase()}:${DOCKER_TAG.toLowerCase()}"
                        sh "docker push ${dockerImageWithRepo}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                branch 'master'
            }
            steps {
                withKubeCredentials(kubectlCredentials: [[credentialsId: 'k8-token']]) {
                    echo 'Deploying application to Kubernetes...'
                    sh "kubectl apply -f deployment-service.yml"
                }
            }
        }

        stage('Verify Deployment') {
            when {
                branch 'master'
            }
            steps {
                withKubeCredentials(kubectlCredentials: [[credentialsId: 'k8-token']]) {
                    echo 'Verifying deployment...'
                    sh "kubectl get all -n auth"
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
