pipeline {
    agent any

    tools {
        // ⚠️ ATTENTION : Assure-toi que ton Maven s'appelle bien "M2_HOME" 
        // dans "Administrer Jenkins > Global Tool Configuration". 
        // Sinon, remplace par le nom que tu as donné (ex: "Maven").
        maven 'M2_HOME' 
    }

    stages {
        stage('Checkout') {
            steps {
                // Récupération du code depuis ton GitHub
                git branch: 'main', url: 'https://github.com/djappa001/JNKS.git'
            }
        }

        stage('Build & Test') {
            steps {
                // Compile et lance les tests unitaires (+ génération rapport Jacoco)
                sh 'mvn clean test'
            }
        }

        // ✅ C'EST L'ÉTAPE QUE TU DOIS AJOUTER POUR LE DEVOIR
         stage('SonarQube Analysis') {
            steps {
                // "SonarQube" doit être le nom donné dans System > SonarQube Servers
                withSonarQubeEnv('SonarQube') { 
                    // Cette commande analyse et envoie le rapport au serveur 9000
                    sh 'mvn sonar:sonar'
                }
            }
        }
       

        stage('Code Packaging'){
            steps{
                // Crée le fichier .jar final
                sh 'mvn package -DskipTests' 
            }
        }
    }

    post {
        success {
            echo "✅ Build et Analyse réussis !"
            // Publie les résultats des tests dans Jenkins
            junit 'target/surefire-reports/*.xml'
        }
        failure {
            echo "❌ Le Build a échoué !"
        }
    }
}
