pipeline {

    agent any

    stages {

        stage('Creation env') {
            steps {
                sh '''
                kubectl get ns dev || kubectl create ns dev
                kubectl get ns qa || kubectl create ns qa
                kubectl get ns staging || kubectl create ns staging
                kubectl get ns prod || kubectl create ns prod
                '''
            }
        }

        stage('Connexion Docker') {
            environment {
                DOCKER_HUB_PASS = credentials('DOCKER_HUB_PASS')
            }

            steps {
                sh '''
                echo $DOCKER_HUB_PASS | docker login \
                -u abzh29 \
                --password-stdin
                '''
            }
        }

        stage('Build Docker') {
            steps {
                sh '''
                docker build -t abzh29/movie:latest movie-service/
                docker build -t abzh29/cast:latest cast-service/
                '''
            }
        }

        stage('Push Docker') {
            steps {
                sh '''
                docker push abzh29/movie:latest
                docker push abzh29/cast:latest
                '''
            }
        }

        stage('Tests') {
            steps {
                sh '''
                helm lint ./charts

                docker images | grep movie
                docker images | grep cast

                kubectl get ns
                '''
            }
        }

        stage('Deploy prod') {
            steps {
                sh '''
                helm upgrade --install movie ./charts -n prod

                kubectl get pods -n prod
                kubectl get svc -n prod
                '''
            }
        }
    }
}
