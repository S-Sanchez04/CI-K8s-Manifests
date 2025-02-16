pipeline {
    agent any

    environment {
        APP_NAME  = "ci-jenkins"
        CHART_DIR = "helm-charts"
        REPO_URL  = "https://S-Sanchez04.github.io/helm-repo"
        GIT_REPO  = "https://github.com/S-Sanchez04/helm-repo.git"
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the current repository; assumes the deployment file is in the root directory.
                checkout scm
            }
        }

        stage('Obtener nuevo tag de la imagen') {
            steps {
                script {
                    // Extract the tag from the deployment file.
                    // Expected format in deployment.yaml: image: ssanchez04/ci-jenkins:1.5
                    def nuevoTag = sh(
                        script: "grep 'image:' deployment.yaml | awk -F ':' '{print \$3}'",
                        returnStdout: true
                    ).trim()
                    env.NEW_IMAGE_TAG = nuevoTag
                    echo "Nuevo tag de la imagen: ${env.NEW_IMAGE_TAG}"
                }
            }
        }

        stage('Generar Helm Chart') {
            steps {
                script {
                    // Create a new Helm chart
                    sh "helm create ${CHART_DIR}"
                    
                    // Update values.yaml with the new image tag
                    sh "sed -i 's/tag:.*/tag: \"${env.NEW_IMAGE_TAG}\"/' ${CHART_DIR}/values.yaml"
                    
                    // Update Chart version using the Jenkins BUILD_NUMBER
                    sh "sed -i 's/version:.*/version: \"1.0.${BUILD_NUMBER}\"/' ${CHART_DIR}/Chart.yaml"
                }
            }
        }

        stage('Empaquetar Chart') {
            steps {
                script {
                    // Package the Helm chart
                    sh "helm package ${CHART_DIR}"
                }
            }
        }

        stage('Subir Chart al Repositorio') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    script {
                        sh """
                            git clone ${GIT_REPO} helm-repo
                            mv ${APP_NAME}-*.tgz helm-repo/
                            cd helm-repo
                            helm repo index .
                            git add .
                            git commit -m 'Agregando nueva versión de ${APP_NAME}'
                            git push origin main
                        """
                    }
                }
            }
        }
    }
}
