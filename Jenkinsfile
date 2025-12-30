// Jenkins Declarative Pipeline for Docker build/push + Helm deploy to multiple namespaces
// Notes:
// - Uses Docker Hub password stored as Jenkins credential "DOCKER_HUB_PASS" (Secret text)
// - Uses Kubernetes kubeconfig stored as Jenkins credential "config" (Secret file)
// - Uses withCredentials(file(...)) to ensure KUBECONFIG points to a real kubeconfig file
// - Adds a quick connectivity check before each Helm deployment

pipeline {
    agent any

    environment {
        DOCKER_ID  = "mkrhov"
        REPO_NAME  = "jenkins-exam-review"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }

    stages {

        stage('Docker Hub Login') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_ID" --password-stdin'
            }
        }

        stage('Build & Push Cast Service') {
            steps {
                dir('cast-service') {
                    sh '''
                        set -e
                        docker build -t $DOCKER_ID/$REPO_NAME:cast-$DOCKER_TAG .
                        docker push $DOCKER_ID/$REPO_NAME:cast-$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Build & Push Movie Service') {
            steps {
                dir('movie-service') {
                    sh '''
                        set -e
                        docker build -t $DOCKER_ID/$REPO_NAME:movie-$DOCKER_TAG .
                        docker push $DOCKER_ID/$REPO_NAME:movie-$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Deploy to dev') {
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                    script {
                        // Quick cluster connectivity and context validation
                        sh '''
                            set -e
                            echo "=== Kubeconfig file path ==="
                            ls -la "$KUBECONFIG"

                            echo "=== Contexts ==="
                            kubectl config get-contexts

                            echo "=== Current context ==="
                            kubectl config current-context

                            echo "=== Cluster info ==="
                            kubectl cluster-info

                            echo "=== Namespaces ==="
                            kubectl get ns
                        '''

                        // Write Helm values for dev
                        writeFile file: 'cast-values-dev.yml', text: """
replicaCount: 1
image:
  repository: ${DOCKER_ID}/${REPO_NAME}
  pullPolicy: Always
  tag: "cast-${DOCKER_TAG}"
imagePullSecrets:
  - name: "dockerhub-secret"
service:
  type: ClusterIP
  port: 80
  targetPort: 8000
"""
                        writeFile file: 'movie-values-dev.yml', text: """
replicaCount: 1
image:
  repository: ${DOCKER_ID}/${REPO_NAME}
  pullPolicy: Always
  tag: "movie-${DOCKER_TAG}"
imagePullSecrets:
  - name: "dockerhub-secret"
service:
  type: ClusterIP
  port: 80
  targetPort: 8000
"""

                        // Deploy with Helm into namespace dev
                        sh 'helm upgrade --install cast-app charts/ -f cast-values-dev.yml -n dev --create-namespace'
                        sh 'helm upgrade --install movie-app charts/ -f movie-values-dev.yml -n dev --create-namespace'
                    }
                }
            }
        }

        stage('Deploy to qa') {
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                    script {
                        // Quick cluster connectivity and context validation
                        sh '''
                            set -e
                            echo "=== Contexts ==="
                            kubectl config get-contexts
                            echo "=== Current context ==="
                            kubectl config current-context
                            echo "=== Cluster info ==="
                            kubectl cluster-info
                        '''

                        // Write Helm values for qa
                        writeFile file: 'cast-values-qa.yml', text: """
replicaCount: 1
image:
  repository: ${DOCKER_ID}/${REPO_NAME}
  pullPolicy: Always
  tag: "cast-${DOCKER_TAG}"
imagePullSecrets:
  - name: "dockerhub-secret"
service:
  type: ClusterIP
  port: 80
  targetPort: 8000
"""
                        writeFile file: 'movie-values-qa.yml', text: """
replicaCount: 1
image:
  repository: ${DOCKER_ID}/${REPO_NAME}
  pullPolicy: Always
  tag: "movie-${DOCKER_TAG}"
imagePullSecrets:
  - name: "dockerhub-secret"
service:
  type: ClusterIP
  port: 80
  targetPort: 8000
"""

                        // Deploy with Helm into namespace qa
                        sh 'helm upgrade --install cast-app charts/ -f cast-values-qa.yml -n qa --create-namespace'
                        sh 'helm upgrade --install movie-app charts/ -f movie-values-qa.yml -n qa --create-namespace'
                    }
                }
            }
        }

        stage('Deploy to staging') {
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                    script {
                        // Quick cluster connectivity and context validation
                        sh '''
                            set -e
                            echo "=== Contexts ==="
                            kubectl config get-contexts
                            echo "=== Current context ==="
                            kubectl config current-context
                            echo "=== Cluster info ==="
                            kubectl cluster-info
                        '''

                        // Write Helm values for staging
                        writeFile file: 'cast-values-staging.yml', text: """
replicaCount: 1
image:
  repository: ${DOCKER_ID}/${REPO_NAME}
  pullPolicy: Always
  tag: "cast-${DOCKER_TAG}"
imagePullSecrets:
  - name: "dockerhub-secret"
service:
  type: ClusterIP
  port: 80
  targetPort: 8000
"""
                        writeFile file: 'movie-values-staging.yml', text: """
replicaCount: 1
image:
  repository: ${DOCKER_ID}/${REPO_NAME}
  pullPolicy: Always
  tag: "movie-${DOCKER_TAG}"
imagePullSecrets:
  - name: "dockerhub-secret"
service:
  type: ClusterIP
  port: 80
  targetPort: 8000
"""

                        // Deploy with Helm into namespace staging
                        sh 'helm upgrade --install cast-app charts/ -f cast-values-staging.yml -n staging --create-namespace'
                        sh 'helm upgrade --install movie-app charts/ -f movie-values-staging.yml -n staging --create-namespace'
                    }
                }
            }
        }

        stage('Deploy to prod') {
            steps {
                // Manual approval gate before production deployment
                timeout(time: 15, unit: 'MINUTES') {
                    input(message: 'Allow deployment to PRODUCTION?', ok: 'Deploy')
                }

                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                    script {
                        // Quick cluster connectivity and context validation
                        sh '''
                            set -e
                            echo "=== Contexts ==="
                            kubectl config get-contexts
                            echo "=== Current context ==="
                            kubectl config current-context
                            echo "=== Cluster info ==="
                            kubectl cluster-info
                        '''

                        // Write Helm values for prod
                        writeFile file: 'cast-values-prod.yml', text: """
replicaCount: 2
image:
  repository: ${DOCKER_ID}/${REPO_NAME}
  pullPolicy: Always
  tag: "cast-${DOCKER_TAG}"
imagePullSecrets:
  - name: "dockerhub-secret"
service:
  type: ClusterIP
  port: 80
  targetPort: 8000
"""
                        writeFile file: 'movie-values-prod.yml', text: """
replicaCount: 2
image:
  repository: ${DOCKER_ID}/${REPO_NAME}
  pullPolicy: Always
  tag: "movie-${DOCKER_TAG}"
imagePullSecrets:
  - name: "dockerhub-secret"
service:
  type: ClusterIP
  port: 80
  targetPort: 8000
"""

                        // Deploy with Helm into namespace prod
                        sh 'helm upgrade --install cast-app charts/ -f cast-values-prod.yml -n prod --create-namespace'
                        sh 'helm upgrade --install movie-app charts/ -f movie-values-prod.yml -n prod --create-namespace'
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'EXAM PASSED!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            // Cleanup generated files in the workspace
            sh 'rm -f cast-values-*.yml movie-values-*.yml || true'
        }
    }
}

