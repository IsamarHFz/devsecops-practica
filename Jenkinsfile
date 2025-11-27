pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Instalar Python y dependencias') {
            steps {
                sh '''
                    python --version || true
                    pip install -r requirements.txt
                '''
            }
        }

        // 1. ANÁLISIS SAST CON SEMGREP
        stage('SAST - Semgrep') {
            steps {
                sh '''
                    pip install semgrep
                    semgrep --config auto .
                '''
            }
        }

        // 2. DEPENDENCIAS CON SAFETY
        stage('Dependencias - Safety') {
            steps {
                sh '''
                    pip install safety
                    safety check --full-report
                '''
            }
        }

        // 3. INSTALAR TRIVY
        stage('Instalar Trivy') {
            steps {
                sh '''
                    sudo apt-get update
                    sudo apt-get install -y wget
                    wget https://github.com/aquasecurity/trivy/releases/latest/download/trivy_0.51.1_Linux-64bit.deb
                    sudo dpkg -i trivy_0.51.1_Linux-64bit.deb || true
                '''
            }
        }

        // 4. SCAN A ARCHIVOS
        stage('Trivy FS Scan') {
            steps {
                sh '''
                    trivy fs .
                '''
            }
        }

        // 5. CONSTRUIR DOCKER IMAGE
        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t devsecops-jenkins .
                '''
            }
        }

        // 6. ANALIZAR IMAGEN DOCKER
        stage('Trivy Image Scan') {
            steps {
                sh '''
                    trivy image devsecops-jenkins
                '''
            }
        }
    }
}
