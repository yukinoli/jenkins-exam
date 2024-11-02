pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    echo 'Construction des images avec docker-compose...'
                    sh 'docker-compose build'
                }
            }
        }
        stage('Test') {
            steps {
                echo 'C\'est parti pour les tests...'
                // Ajoute tes étapes de test ici
            }
        }
        stage('Deploy') {
            when {
                branch 'master'
            }
            steps {
                echo 'Déploiement sur la branche master uniquement !'
                sh 'docker-compose up -d' // Déployer les services
            }
        }
    }
}
