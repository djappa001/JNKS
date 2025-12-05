pipeline {
    agent any

    tools {
        maven 'M2_HOME' 
        jdk 'JAVA_HOME'
    }

    environment {
        // Définit le nom de l'image pour éviter de le répéter partout
        IMAGE_NAME = "yappa01/student-app"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/djappa001/JNKS.git'
            }
        }

        // CORRECTION 0% COUVERTURE :
        // On utilise 'verify' au lieu de 'test' pour générer le rapport JaCoCo
        // Et on lance sonar tout de suite après pour qu'il trouve le rapport.
        stage('Build, Test & Analyze') {
            steps {
                withSonarQubeEnv('SonarQube') { 
                    sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=student-management -Dsonar.projectName="Student App"'
                }
            }
        }
        
        // Attente de la validation du Quality Gate (Optionnel mais recommandé)
        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
       
        stage('Code Packaging'){
            steps{
                // Crée le .jar sans relancer les tests (déjà faits avant)
                sh 'mvn package -DskipTests' 
            }
        }

        // --- ÉTAPE 1 : DOCKER BUILD (Construction seulement) ---
        stage('Docker Build') {
            steps {
                script {
                    echo "🔨 Construction de l'image Docker..."
                    // Construction avec le numéro de build
                    sh "docker build -t ${IMAGE_NAME}:${env.BUILD_NUMBER} ."
                    
                    // Tag de la version 'latest' en local
                    sh "docker tag ${IMAGE_NAME}:${env.BUILD_NUMBER} ${IMAGE_NAME}:latest"
                }
            }
        }

        // --- ÉTAPE 2 : DOCKER PUSH (Envoi seulement) ---
        stage('Docker Push') {
            steps {
                script {
                    // On ne récupère les identifiants QUE pour le push
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-id', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        echo "📤 Connexion et Envoi vers Docker Hub..."
                        
                        // 1. Login
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        
                        // 2. Push de la version précise
                        sh "docker push ${IMAGE_NAME}:${env.BUILD_NUMBER}"
                        
                        // 3. Push de la version latest
                        sh "docker push ${IMAGE_NAME}:latest"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline terminé avec succès !"
            junit 'target/surefire-reports/*.xml'
        }
        failure {
            echo "❌ Le pipeline a échoué."
        }
        always {
            // Nettoyage de l'espace disque (supprime les images locales après le push)
            sh "docker rmi ${IMAGE_NAME}:${env.BUILD_NUMBER} || true"
            sh "docker rmi ${IMAGE_NAME}:latest || true"
        }
    }
}
