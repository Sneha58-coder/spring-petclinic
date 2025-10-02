pipeline {
    agent any

    environment {
        // Azure Container Registry
        ACR_NAME = "luckyregistry123"
        ACR_LOGIN_SERVER = "luckyregistry123.azurecr.io"
        // Docker image name
        IMAGE_NAME = "spring-petclinic"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        // Kubernetes namespace
        K8S_NAMESPACE = "default"
        // Trivy scan result path
        TRIVY_REPORT = "trivy-report.json"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Sneha58-coder/spring-petclinic.git'
            }
        }

        stage('Maven Validate') {
            steps {
                sh './mvnw validate'
            }
        }

        stage('Maven Compile') {
            steps {
                sh './mvnw compile'
            }
        }

        stage('Maven Test') {
            steps {
                sh './mvnw test'
            }
        }

        stage('Maven Package') {
            steps {
                sh './mvnw package -DskipTests'
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'SONARCLOUD_CREDS', passwordVariable: 'SONAR_TOKEN', usernameVariable: 'SONAR_USER')]) {
                    sh "./mvnw sonar:sonar -Dsonar.login=$SONAR_TOKEN"
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
                sh "trivy image --exit-code 1 --format json -o ${TRIVY_REPORT} ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG} || true"
            }
        }

        stage('Login to ACR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'AZURE_ACR_CREDS', usernameVariable: 'ACR_USER', passwordVariable: 'ACR_PASS')]) {
                    sh "docker login ${ACR_LOGIN_SERVER} -u ${ACR_USER} -p ${ACR_PASS}"
                }
            }
        }

        stage('Push Image to ACR') {
            steps {
                sh "docker push ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl set image deployment/spring-petclinic-deployment spring-petclinic=${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG} -n ${K8S_NAMESPACE} || kubectl apply -f k8s/deployment.yaml -n ${K8S_NAMESPACE}"
            }
        }

        stage('Test Deployment') {
            steps {
                sh "kubectl get pods -n ${K8S_NAMESPACE}"
                sh "kubectl get svc -n ${K8S_NAMESPACE}"
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: "${TRIVY_REPORT}", allowEmptyArchive: true
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check the logs!"
        }
    }
}

