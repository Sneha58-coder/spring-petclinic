pipeline {
    agent any

    environment {
        // SonarCloud credentials ID (from Jenkins > Credentials)
        SONAR_CLOUD = 'sonarcloud-creds'  
        // Azure container registry credentials ID
        ACR_CREDENTIALS = 'acr-creds'  
        // Azure registry login server
        ACR_LOGIN_SERVER = 'luckyregistry123.azurecr.io'  
        // Kubernetes namespace (optional)
        KUBE_NAMESPACE = 'default'  
        // Docker image name
        IMAGE_NAME = 'spring-petclinic'  
        // Tag for Docker image
        IMAGE_TAG = 'latest'  
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Sneha58-coder/spring-petclinic.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn validate'
                sh 'mvn compile'
                sh 'mvn test'
                sh 'mvn package'
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${SONAR_CLOUD}", usernameVariable: 'SONAR_USER', passwordVariable: 'SONAR_PASS')]) {
                    sh "mvn sonar:sonar -Dsonar.login=$SONAR_PASS"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Scan Image with Trivy') {
            steps {
                sh "trivy image ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Push to ACR') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${ACR_CREDENTIALS}", usernameVariable: 'ACR_USER', passwordVariable: 'ACR_PASS')]) {
                    sh "docker login ${ACR_LOGIN_SERVER} -u $ACR_USER -p $ACR_PASS"
                    sh "docker push ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // Apply Kubernetes manifests from your repo
                sh "kubectl apply -f k8s/"
            }
        }

        stage('Test Deployment') {
            steps {
                sh "kubectl get pods -n ${KUBE_NAMESPACE}"
                sh "kubectl get svc -n ${KUBE_NAMESPACE}"
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}

