pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    docker.build('ton-utilisateur/ton-image')
                }
            }
        }
        stage('Test') {
            steps {
                echo 'C\'est parti pour les tests...'
            }
        }
        stage('Deploy') {
            when {
                branch 'master'
            }
            steps {
                echo 'Déploiement sur la branche master uniquement !'
            }
        }
    }
}
