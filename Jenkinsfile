pipeline {
    agent any
    environment {
        GIT_REPO = 'https://github.com/amitopenwriteup/upgrad.git'
        GIT_BRANCH = 'main'
        DOCKER_REGISTRY = 'localhost:5000'
        IMAGE_NAME = 'myimage'
        IMAGE_TAG = 'latest'
        DOCKERFILE_PATH = 'dockerfile'
    }
    stages {
        stage('Checkout the Git repository') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO}"
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def dockerImage = docker.build("${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}", "-f ${DOCKERFILE_PATH} .")
                    dockerImage.push()
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    // Run Trivy scan and output results to a file
                    def scanResult = sh(script: "trivy image --exit-code 1 --severity HIGH,CRITICAL ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} | tee trivy_report.txt", returnStatus: true)
                    
                    // Check if vulnerabilities were found
                    if (scanResult == 1) {
                        error "Trivy scan found HIGH/CRITICAL vulnerabilities. Review the 'trivy_report.txt' log for details."
                    }
                }
            }
        }
        stage('Deploy or Push to Production') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                echo "No critical vulnerabilities found, proceeding with deployment."
                sh 'docker run -dit -p 8000:8000 localhost:5010/myimage'
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'trivy_report.txt', fingerprint: true
        }
        failure {
            echo "Pipeline failed due to vulnerabilities. Check 'trivy_report.txt'."
        }
    }
}
