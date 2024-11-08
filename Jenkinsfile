pipeline {
    agent any
    environment {
        DOCKER_ID = "yukino" // replace this with your docker-id
        DOCKER_IMAGE = "datascientestapi"
        DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
        KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config saved on Jenkins
    }
    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
                echo 'Code source récupéré depuis GitHub.'
                echo "GIT_BRANCH is: ${env.GIT_BRANCH}"
                echo "BRANCH_NAME is: ${env.BRANCH_NAME}"
            }
        }
        stage('Docker Build') { // docker-compose build image stage
            steps {
                script {
                    sh '''
                        docker rm -f jenkins || true
                        docker-compose build
                        sleep 6
                        # Tagging the image after build
                        docker tag pipeline-exam-jenkins_movie_service $DOCKER_ID/${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
                }
            }
        }
        stage('Docker Run') { // run container from our built image
            steps {
                script {
                    sh '''
                        docker-compose up -d
                        sleep 10
                    '''
                }
            }
        }
        stage('Test Acceptance') { // validate that the container responds to the request
            steps {
                script {
                    sh 'curl localhost'
                }
            }
        }
        stage('Docker Push') { // push the built image to Docker Hub
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve docker password from secret text called docker_hub_pass saved on Jenkins
            }
            steps {
                script {
                    sh '''
                        docker login -u $DOCKER_ID -p $DOCKER_PASS
                        docker push $DOCKER_ID/${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
                }
            }
        }
        stage('Deploiement en dev') {
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        kubectl apply -f nginx.yaml --namespace dev
                    '''
                }
            }
        }
        stage('Deploiement en QA') {
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        kubectl apply -f nginx.yaml --namespace qa
                    '''
                }
            }
        }
        stage('Deploiement en staging') {
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        kubectl apply -f nginx.yaml --namespace staging
                    '''
                }
            }
        }
        stage('Deploiement en prod') {
            when {
                expression {
                    return env.BRANCH_NAME == 'master' || env.BRANCH_NAME == 'origin/master' || env.GIT_BRANCH == 'master' || env.GIT_BRANCH == 'origin/master'
                }
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Voulez-vous procéder au déploiement en production ?', ok: 'Oui'
                }
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        kubectl apply -f nginx.yaml --namespace prod
                    '''
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline terminé avec succès.'
        }
        failure {
            echo 'Le pipeline a échoué, veuillez vérifier les logs.'
        }
    }
}
