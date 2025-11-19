pipeline {
    agent any

    triggers {
        pollSCM('H/5 * * * *')
    }

    environment {
        /* Définition du nom du tag. Ici : v1.0. suivi du numéro de build Jenkins */
        TAG_NAME = "v1.0.${BUILD_NUMBER}"
        /* Mettez l'URL de votre repo SANS le 'https://' */
        REPO_URL_BARE = 'github.com/lorenzojg/HelloWorldMaven.git'
    }

    stages {
        stage('Récupération du Code') {
            steps {
                git branch: 'master',
                    credentialsId: '46732823-874a-4be8-990e-3b669d3c4dbe',
                    url: "https://${REPO_URL_BARE}"
            }
        }
        
        stage('Build') {
            steps {
                script {
                    echo "Démarrage de la construction..."
                    sh 'mvn clean package'
                }
            }
        }
        
        stage('Création & Push du Tag') {
            steps {
                /* Utilisation du bloc withCredentials pour récupérer login/pass */
                withCredentials([usernamePassword(credentialsId: '46732823-874a-4be8-990e-3b669d3c4dbe', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    
                    sh """
                        # 1. Configuration de l'identité locale pour le commit/tag
                        git config user.email "lorenzojg@hotmail.fr"
                        git config user.name "Lorenzo JACCAUD--GODEFROY"

                        # 2. Création du tag localement
                        git tag -a ${TAG_NAME} -m "Version générée automatiquement par Jenkins Build #${BUILD_NUMBER}"

                        # 3. Push du tag vers le repo distant
                        # On injecte les credentials dans l'URL pour l'authentification
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@${REPO_URL_BARE} ${TAG_NAME}
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            sh "echo 'Succès'"
        }
    }
}
