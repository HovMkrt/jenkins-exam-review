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
            environment {
                KUBECONFIG_CRED = credentials("config")
            }
            steps {
                script {
                    sh '''
                        set -e
                        rm -rf .kube
                        mkdir -p .kube
                        cat "$KUBECONFIG_CRED" > .kube/config
                    '''

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
  targetPort: 8001
"""

                    sh 'helm upgrade --install cast-app charts/ -f cast-values-dev.yml -n dev --create-namespace'
                    sh 'helm upgrade --install movie-app charts/ -f movie-values-dev.yml -n dev --create-namespace'
                }
            }
        }

        stage('Deploy to qa') {
            environment {
                KUBECONFIG_CRED = credentials("config")
            }
            steps {
                script {
                    sh '''
                        set -e
                        rm -rf .kube
                        mkdir -p .kube
                        cat "$KUBECONFIG_CRED" > .kube/config
                    '''

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
  targetPort: 8001
"""

                    sh 'helm upgrade --install cast-app charts/ -f cast-values-qa.yml -n qa --create-namespace'
                    sh 'helm upgrade --install movie-app charts/ -f movie-values-qa.yml -n qa --create-namespace'
                }
            }
        }

        stage('Deploy to staging') {
            environment {
                KUBECONFIG_CRED = credentials("config")
            }
            steps {
                script {
                    sh '''
                        set -e
                        rm -rf .kube
                        mkdir -p .kube
                        cat "$KUBECONFIG_CRED" > .kube/config
                    '''

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
  targetPort: 8001
"""

                    sh 'helm upgrade --install cast-app charts/ -f cast-values-staging.yml -n staging --create-namespace'
                    sh 'helm upgrade --install movie-app charts/ -f movie-values-staging.yml -n staging --create-namespace'
                }
            }
        }

        stage('Deploy to prod') {
            environment {
                KUBECONFIG_CRED = credentials("config")
            }
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input(message: 'Разрешить деплой в PRODUCTION?', ok: 'Да')
                }

                script {
                    sh '''
                        set -e
                        rm -rf .kube
                        mkdir -p .kube
                        cat "$KUBECONFIG_CRED" > .kube/config
                    '''

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
  targetPort: 8001
"""

                    sh 'helm upgrade --install cast-app charts/ -f cast-values-prod.yml -n prod --create-namespace'
                    sh 'helm upgrade --install movie-app charts/ -f movie-values-prod.yml -n prod --create-namespace'
                }
            }
        }
    }

    post {
        success {
            echo 'ЭКЗАМЕН СДАН!'
        }
        failure {
            echo 'Ошибка!'
        }
        always {
            sh 'rm -rf .kube cast-values-*.yml movie-values-*.yml || true'
        }
    }
}

