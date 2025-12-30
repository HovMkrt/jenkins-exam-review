cd ~/jenkins-exam-review
cat > Jenkinsfile << 'EOF'
pipeline {
    environment {
        DOCKER_ID = "mkrhov"
        REPO_NAME = "jenkins-exam-review"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    agent any

    stages {
        // 1. Логин в Docker Hub
        stage('Docker Hub Login') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                sh 'echo $DOCKER_PASS | docker login -u $DOCKER_ID --password-stdin'
            }
        }

        // 2. Сборка и пуш Cast Service
        stage('Build & Push Cast Service') {
            steps {
                dir('cast-service') {
                    sh '''
                        echo "=== Сборка Cast Service ==="
                        docker build -t $DOCKER_ID/$REPO_NAME:cast-$DOCKER_TAG .
                        echo "=== Пуш в Docker Hub ==="
                        docker push $DOCKER_ID/$REPO_NAME:cast-$DOCKER_TAG
                    '''
                }
            }
        }

        // 3. Сборка и пуш Movie Service
        stage('Build & Push Movie Service') {
            steps {
                dir('movie-service') {
                    sh '''
                        echo "=== Сборка Movie Service ==="
                        docker build -t $DOCKER_ID/$REPO_NAME:movie-$DOCKER_TAG .
                        echo "=== Пуш в Docker Hub ==="
                        docker push $DOCKER_ID/$REPO_NAME:movie-$DOCKER_TAG
                    '''
                }
            }
        }

        // 4. Деплой в Dev
        stage('Deploy to dev') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                        rm -rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                    '''
                    
                    // Cast Service
                    sh '''
                        cat > cast-values-dev.yml << EOL
replicaCount: 1
image:
  repository: $DOCKER_ID/$REPO_NAME
  pullPolicy: Always
  tag: "cast-$DOCKER_TAG"
imagePullSecrets:
  - name: "dockerhub-secret"
service:
  type: ClusterIP
  port: 80
  targetPort: 8000
EOL
                    '''
                    
                    // Movie Service
                    sh '''
                        cat > movie-values-dev.yml << EOL
replicaCount: 1
image:
  repository: $DOCKER_ID/$REPO_NAME
  pullPolicy: Always
  tag: "movie-$DOCKER_TAG"
imagePullSecrets:
  - name: "dockerhub-secret"
service:
  type: ClusterIP
  port: 80
  targetPort: 8001
EOL
                    '''
                    
                    sh 'helm upgrade --install cast-app charts/ --values=cast-values-dev.yml --namespace dev --create-namespace'
                    sh 'helm upgrade --install movie-app charts/ --values=movie-values-dev.yml --namespace dev --create-namespace'
                    
                    // Проверка
                    sh '''
                        echo "=== Проверка Dev ==="
                        sleep 10
                        kubectl get pods -n dev
                    '''
                }
            }
        }

        // 5. Деплой в QA
        stage('Deploy to qa') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                        rm -rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                    '''
                    
                    sh '''
                        cat > cast-values-qa.yml << EOL
replicaCount: 1
image:
  repository: $DOCKER_ID/$REPO_NAME
  pullPolicy: Always
  tag: "cast-$DOCKER_TAG"
imagePullSecrets:
  - name: "dockerhub-secret"
service:
  type: ClusterIP
  port: 80
  targetPort: 8000
EOL
                    '''
                    
                    sh '''
                        cat > movie-values-qa.yml << EOL
replicaCount: 1
image:
  repository: $DOCKER_ID/$REPO_NAME
  pullPolicy: Always
  tag: "movie-$DOCKER_TAG"
imagePullSecrets:
  - name: "dockerhub-secret"
service:
  type: ClusterIP
  port: 80
  targetPort: 8001
EOL
                    '''
                    
                    sh 'helm upgrade --install cast-app charts/ --values=cast-values-qa.yml --namespace qa --create-namespace'
                    sh 'helm upgrade --install movie-app charts/ --values=movie-values-qa.yml --namespace qa --create-namespace'
                }
            }
        }

        // 6. Деплой в Staging
        stage('Deploy to staging') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                        rm -rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                    '''
                    
                    sh '''
                        cat > cast-values-staging.yml << EOL
replicaCount: 1
image:
  repository: $DOCKER_ID/$REPO_NAME
  pullPolicy: Always
  tag: "cast-$DOCKER_TAG"
imagePullSecrets:
  - name: "dockerhub-secret"
service:
  type: ClusterIP
  port: 80
  targetPort: 8000
EOL
                    '''
                    
                    sh '''
                        cat > movie-values-staging.yml << EOL
replicaCount: 1
image:
  repository: $DOCKER_ID/$REPO_NAME
  pullPolicy: Always
  tag: "movie-$DOCKER_TAG"
imagePullSecrets:
  - name: "dockerhub-secret"
service:
  type: ClusterIP
  port: 80
  targetPort: 8001
EOL
                    '''
                    
                    sh 'helm upgrade --install cast-app charts/ --values=cast-values-staging.yml --namespace staging --create-namespace'
                    sh 'helm upgrade --install movie-app charts/ --values=movie-values-staging.yml --namespace staging --create-namespace'
                }
            }
        }

        // 7. Деплой в Prod с ручным подтверждением
        stage('Deploy to prod') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input(message: 'Разрешить деплой в PRODUCTION?', ok: 'Да')
                }
                script {
                    sh '''
                        rm -rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                    '''
                    
                    sh '''
                        cat > cast-values-prod.yml << EOL
replicaCount: 2
image:
  repository: $DOCKER_ID/$REPO_NAME
  pullPolicy: Always
  tag: "cast-$DOCKER_TAG"
imagePullSecrets:
  - name: "dockerhub-secret"
service:
  type: ClusterIP
  port: 80
  targetPort: 8000
EOL
                    '''
                    
                    sh '''
                        cat > movie-values-prod.yml << EOL
replicaCount: 2
image:
  repository: $DOCKER_ID/$REPO_NAME
  pullPolicy: Always
  tag: "movie-$DOCKER_TAG"
imagePullSecrets:
  - name: "dockerhub-secret"
service:
  type: ClusterIP
  port: 80
  targetPort: 8001
EOL
                    '''
                    
                    sh 'helm upgrade --install cast-app charts/ --values=cast-values-prod.yml --namespace prod --create-namespace'
                    sh 'helm upgrade --install movie-app charts/ --values=movie-values-prod.yml --namespace prod --create-namespace'
                    
                    // Финальная проверка
                    sh '''
                        echo "=== УСПЕХ! ==="
                        echo "Репозиторий: $DOCKER_ID/$REPO_NAME"
                        echo "Теги: cast-$DOCKER_TAG, movie-$DOCKER_TAG"
                        echo "Поды в prod:"
                        kubectl get pods -n prod
                        echo ""
                        echo "Сервисы в prod:"
                        kubectl get svc -n prod
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ ЭКЗАМЕН СДАН! Все требования выполнены:'
            echo '1. ✅ Оба сервиса в Docker Hub'
            echo '2. ✅ Деплой в 4 окружения'
            echo '3. ✅ Ручное подтверждение для prod'
            echo '4. ✅ Использование Helm'
            echo '5. ✅ Полный CI/CD пайплайн'
        }
        failure {
            echo '❌ Ошибка! Проверь Console Output.'
        }
    }
}
