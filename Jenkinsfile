pipeline {
    agent any

    environment {
        APP_NAME   = "ci-jenkins"
        CHART_DIR  = "helm-charts"
        REPO_URL   = "https://S-Sanchez04.github.io/helm-repo"
        GIT_REPO   = "https://github.com/S-Sanchez04/helm-repo.git"
        HAS_CHANGES = "false"  // flag to track if the deployment file changed
    }

    stages {
        stage('Clonar repo de manifiestos') {
            steps {
                git branch: 'main', credentialsId: 'github-credentials', url: 'https://github.com/S-Sanchez04/CI-K8s-Manifests.git'
            }
        }

        stage('Verificar si api-deployment.yaml cambió') {
            steps {
                script {
                    def changes = sh(
                        script: "git diff --name-only HEAD~1 HEAD | grep 'k8s/api-deployment.yaml' || true", 
                        returnStdout: true
                    ).trim()

                    if (changes == '') {
                        echo "No hubo cambios en api-deployment.yaml, terminando el pipeline."
                        env.HAS_CHANGES = "false"
                    } else {
                        echo "api-deployment.yaml ha cambiado, continuando con el pipeline..."
                        env.HAS_CHANGES = "true"
                    }
                }
            }
        }

        stage('Obtener nuevo tag de la imagen') {
            when {
                expression { env.HAS_CHANGES == "true" }
            }
            steps {
                script {
                    def nuevoTag = sh(
                        script: "grep 'image:' k8s/api-deployment.yaml | awk -F ':' '{print \$3}'",
                        returnStdout: true
                    ).trim()
                    env.NEW_IMAGE_TAG = nuevoTag
                    echo "Nuevo tag de la imagen: ${env.NEW_IMAGE_TAG}"
                }
            }
        }

        stage('Generar Helm Chart') {
            when {
                expression { env.HAS_CHANGES == "true" }
            }
            steps {
                script {
                    sh "helm create ${CHART_DIR}"

                    // Actualiza values.yaml con el nuevo tag
                    sh "sed -i 's/tag:.*/tag: \"${env.NEW_IMAGE_TAG}\"/' ${CHART_DIR}/values.yaml"

                    // Actualiza la versión del Chart
                    sh "sed -i 's/version:.*/version: \"1.0.${BUILD_NUMBER}\"/' ${CHART_DIR}/Chart.yaml"
                }
            }
        }

        stage('Empaquetar Chart') {
            when {
                expression { env.HAS_CHANGES == "true" }
            }
            steps {
                script {
                    sh "helm package ${CHART_DIR}"
                }
            }
        }

        stage('Subir Chart al Repositorio') {
            when {
                expression { env.HAS_CHANGES == "true" }
            }
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
