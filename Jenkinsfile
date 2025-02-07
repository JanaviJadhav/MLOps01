pipeline { 
    agent any
    environment {
        DOCKERHUB_CREDENTIAL_ID = 'mlops-jenkins-dockerhub-token'
        DOCKERHUB_REGISTRY = 'https://index.docker.io/v1/'
        DOCKERHUB_REPOSITORY = 'janavi31/mlops-proj-01'
        VENV_PATH = 'venv'  // Set Virtual Environment Path
    }
    stages {
        stage('Clone Repository') {
            steps {
                script {
                    echo 'Cloning GitHub Repository...'
                    checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'mlops-git-token', url: 'https://github.com/JanaviJadhav/MLOps01.git']])
                }
            }
        }
        stage('Setup Virtual Environment') {
            steps {
                script {
                    echo 'Setting up Virtual Environment...'
                    sh """
                        python3 -m venv ${VENV_PATH}
                        . ${VENV_PATH}/bin/activate  # Fixed 'source' issue
                        ${VENV_PATH}/bin/pip install --upgrade pip
                        ${VENV_PATH}/bin/pip install -r requirements.txt
                    """
                }
            }
        }
        stage('Lint Code') {
            steps {
                script {
                    echo 'Linting Python Code...'
                    sh """
                        . ${VENV_PATH}/bin/activate
                        ${VENV_PATH}/bin/pylint app.py train.py --output=pylint-report.txt --exit-zero
                        ${VENV_PATH}/bin/flake8 app.py train.py --ignore=E501,E302 --output-file=flake8-report.txt
                        ${VENV_PATH}/bin/black app.py train.py
                    """
                }
            }
        }
        stage('Test Code') {
            steps {
                script {
                    echo 'Testing Python Code...'
                    sh """
                        . ${VENV_PATH}/bin/activate
                        ${VENV_PATH}/bin/pytest tests/
                    """
                }
            }
        }
        stage('Trivy FS Scan') {
            steps {
                script {
                    echo 'Scanning Filesystem with Trivy...'
                    sh "trivy fs ./ --format table -o trivy-fs-report.html"
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker Image...'
                    dockerImage = docker.build("${DOCKERHUB_REPOSITORY}:latest") 
                }
            }
        }
        stage('Trivy Docker Image Scan') {
            steps {
                script {
                    echo 'Scanning Docker Image with Trivy...'
                    sh "trivy image ${DOCKERHUB_REPOSITORY}:latest --format table -o trivy-image-report.html"
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    echo 'Pushing Docker Image to DockerHub...'
                    docker.withRegistry("${DOCKERHUB_REGISTRY}", "${DOCKERHUB_CREDENTIAL_ID}"){
                        dockerImage.push('latest')
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    echo 'Deploying to production...'
                    sh "aws ecs update-service --cluster iquant-ecs --service iquant-ecs-svc --force-new-deployment"
                }
            }
        }
    }
}
