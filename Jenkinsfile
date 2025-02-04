pipeline {
    agent any

    environment {
        PATH = "C:\\Program Files\\Docker\\Docker\\resources\\bin;C:\\Program Files\\IBM\\Cloud\\bin;C:\\Windows\\System32"
        IBM_CLOUD_REGION = 'ap-north'
        IBM_CLOUD_REGISTRY_NAMESPACE = 'config-manage'
        IBM_CLOUD_REGISTRY_URL = 'jp.icr.io'  // IBM Container Registry for Tokyo
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/PriyaKumari-2002/config-manage.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    bat """
                    echo 🔨 Building Docker Image...
                    docker build -t config-manage:latest -f docker/Dockerfile .
                    """
                }
            }
        }

       stage('Push Docker Image') {
    steps {
        script {
            bat 'echo 📦 Preparing to push Docker image...'

            withCredentials([string(credentialsId: 'ibmcloud-api-key', variable: 'IBM_CLOUD_API_KEY')]) {
                bat """
                echo 🔐 Setting IBM Cloud API Endpoint...
                ibmcloud api https://cloud.ibm.com

                echo 🔐 Logging into IBM Cloud...
                ibmcloud login -a https://cloud.ibm.com -u passcode -p DWipsATbNc
                ibmcloud target -r ap-north
                ibmcloud cr login
                """
            }

            def imageName = "${IBM_CLOUD_REGISTRY_URL}/${IBM_CLOUD_REGISTRY_NAMESPACE}/config-manage:latest"
            bat """
            echo 🚀 Tagging and Pushing Docker Image...
            docker tag config-manage:latest ${imageName}
            docker push ${imageName}
            """
        }
    }
}
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'ibmcloud-api-key', variable: 'IBM_CLOUD_API_KEY')]) {
                        bat """
                        echo ⚙️ Configuring Kubernetes Cluster...
                        ibmcloud ks cluster config --cluster YOUR_CLUSTER_NAME

                        echo 📡 Deploying to Kubernetes...
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
