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
                checkout scm
            }
        }

        stage('Obtener nuevo tag de la imagen') {
            steps {
                script {
                    def nuevoTag = sh(
                        script: "grep 'image:' api-deployment.yaml | awk -F ':' '{print \$3}'",
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
                    sh "helm create ${CHART_DIR}"
                    sh "sed -i 's/tag:.*/tag: \"${env.NEW_IMAGE_TAG}\"/' ${CHART_DIR}/values.yaml"
                    sh "sed -i 's/version:.*/version: \"1.0.${BUILD_NUMBER}\"/' ${CHART_DIR}/Chart.yaml"
                }
            }
        }

        stage('Empaquetar Chart') {
            steps {
                script {
                    sh "helm package ${CHART_DIR}"
                }
            }
        }

        stage('Subir Chart al Repositorio') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    script {
                        sh """
                            rm -rf helm-repo
                            git clone ${GIT_REPO} helm-repo
                            mv ${CHART_DIR}-*.tgz helm-repo/
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
