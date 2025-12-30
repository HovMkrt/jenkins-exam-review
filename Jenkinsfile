pipeline {
    environment {
        DOCKER_ID = "mkrhov"
        APP_IMAGE = "jenkins-exam-review"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    agent any

    stages {
        // 1. Сборка Docker образа
        stage('Build Docker Image') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")  // ИЗМЕНИТЬ ТУТ
            }
            steps {
                dir('cast-service') {
                    sh '''
                        docker login -u $DOCKER_ID -p $DOCKER_PASS
                        docker build -t $DOCKER_ID/$APP_IMAGE:$DOCKER_TAG .
                    '''
                }
            }
        }        
        // 2. Пуш образа в Docker Hub
        stage('Push to Docker Hub') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")  // ИЗМЕНИТЬ ТУТ
            }
            steps {
                sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$APP_IMAGE:$DOCKER_TAG
                '''
            }
        }        
// 3. Деплой в Dev
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
                        cp charts/values.yaml values.yml
                        sed -i "s|tag:.*|tag: $DOCKER_TAG|g" values.yml
                        cat values.yml | grep -A2 -B2 "image:"
                    '''
                    sh 'helm upgrade --install exam-app charts/ --values=values.yml --namespace dev'
                }
            }
        }

        // 4. Деплой в QA
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
                        cp charts/values.yaml values.yml
                        sed -i "s|tag:.*|tag: $DOCKER_TAG|g" values.yml
                    '''
                    sh 'helm upgrade --install exam-app charts/ --values=values.yml --namespace qa'
                }
            }
        }

        // 5. Деплой в Staging
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
                        cp charts/values.yaml values.yml
                        sed -i "s|tag:.*|tag: $DOCKER_TAG|g" values.yml
                    '''
                    sh 'helm upgrade --install exam-app charts/ --values=values.yml --namespace staging'
                }
            }
        }

        // 6. Деплой в Prod (с ручным подтверждением)
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
                        cp charts/values.yaml values.yml
                        sed -i "s|tag:.*|tag: $DOCKER_TAG|g" values.yml
                    '''
                    sh 'helm upgrade --install exam-app charts/ --values=values.yml --namespace prod'
                }
            }
        }
    }

    post {
        failure {
            echo 'Пайплайн упал! Проверь консольный вывод.'
        }
        success {
            echo 'Пайплайн успешно завершен!'
        }
    }
}
