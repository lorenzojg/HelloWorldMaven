// Fichier à placer à la racine de votre dépôt Git sous le nom "Jenkinsfile"

pipeline {
    agent any // S'exécute sur n'importe quel agent Jenkins disponible

    // Définit les outils à utiliser. 
    // 'Maven-3.9' doit être un outil configuré dans 
    // "Manage Jenkins" > "Global Tool Configuration".
    tools {
        maven 'Maven-3.8' 
    }

    triggers {
        // Déclencheur : vérifie le dépôt Git toutes les 5 minutes pour des changements.
        // Si un changement est détecté, le pipeline démarre.
        pollSCM('H/5 * * * *')
    }

    stages {
        // Étape 1 : Récupération du code
        // Cette étape utilise les credentials configurés dans le pipeline Jenkins
        // pour se connecter à votre dépôt Git.
        stage('Checkout') {
            steps {
                checkout scm
                echo "Code source récupéré."
            }
        }

        // Étape 2 : Build avec Maven
        // 'mvn' est disponible car nous l'avons défini dans la section 'tools'.
        stage('Build') {
            steps {
                echo "Lancement du build Maven..."
                // Exécute 'mvn clean install'.
                // Si vous utilisez le wrapper Maven, utilisez : sh './mvnw clean install'
                sh 'mvn clean install'
            }
        }

        // Étape 3 : Tagger et Pousser le tag
        stage('Tag & Push') {
            steps {
                script {
                    // Utilise une variable d'environnement de Jenkins pour le nom du tag
                    def tagName = "build-${env.BUILD_NUMBER}"

                    echo "Préparation du tag : ${tagName}"

                    // Le 'sh' (shell) suivant s'exécute sur l'agent Jenkins.
                    // Il réutilise l'authentification de l'étape 'Checkout'.
                    sh """
                    echo "Configuration de l'identité Git pour ce commit..."
                    # C'est une bonne pratique pour l'audit, pour savoir qui a créé le tag
                    git config user.email "jenkins@votre-domaine.com"
                    git config user.name "Jenkins CI"
                    
                    echo "Création du tag annoté : ${tagName}"
                    # Nous créons un tag annoté (-a) avec un message (-m)
                    git tag -a ${tagName} -m "Tag créé par Jenkins Build ${env.BUILD_NUMBER}"
                    
                    echo "Push du tag vers 'origin'..."
                    # Pousse le tag spécifique vers le dépôt distant
                    git push origin ${tagName}
                    """
                    
                    echo "Tag ${tagName} poussé avec succès."
                }
            }
        }
    }
    
    post {
        // S'exécute toujours à la fin du pipeline
        always {
            echo "Nettoyage de l'espace de travail..."
            // Supprime tous les fichiers pour libérer de l'espace
            cleanWs() 
        }
        // En cas de succès, archive les résultats des tests (bonne pratique)
        success {
            junit 'target/surefire-reports/*.xml'
        }
    }
}
