pipeline {
    agent any

    tools {
        maven 'M2_HOME' 
        // J'ajoute le JDK pour être sûr que tout le monde utilise la bonne version
        jdk 'JAVA_HOME'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/djappa001/JNKS.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') { 
                    sh 'mvn sonar:sonar -Dsonar.projectKey=student-management -Dsonar.projectName="Student App"'
                }
            }
        }
       
        stage('Code Packaging'){
            steps{
                // Crée le fichier .jar dans le dossier target/
                sh 'mvn package -DskipTests' 
            }
        }

        // --- NOUVELLE ÉTAPE AJOUTÉE ---
        stage('Docker Build & Push') {
            steps {
                script {
                    // On récupère les identifiants 'dockerhub-id' configurés dans Jenkins
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-id', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        
                        // 1. Connexion sécurisée à Docker Hub
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        
                        // 2. Construction de l'image avec un Tag unique (numéro du build)
                        sh "docker build -t yappa01/student-app:${env.BUILD_NUMBER} ."
                        
                        // 3. Envoi de l'image versionnée
                        sh "docker push yappa01/student-app:${env.BUILD_NUMBER}"
                        
                        // 4. Mise à jour du tag 'latest' et envoi
                        sh "docker tag yappa01/student-app:${env.BUILD_NUMBER} yappa01/student-app:latest"
                        sh "docker push yappa01/student-app:latest"
                    }
                }
            }
        }
        // -----------------------------
    }

    post {
        success {
            echo "✅ Build, Analyse et Déploiement Docker réussis !"
            junit 'target/surefire-reports/*.xml'
        }
        failure {
            echo "❌ Le pipeline a échoué !"
        }
    }
}
