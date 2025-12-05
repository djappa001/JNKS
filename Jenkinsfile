pipeline {
    agent any

    tools {
        // Assure-toi que ces noms correspondent à ta config dans "Global Tool Configuration"
        maven 'M2_HOME' 
        jdk 'JAVA_HOME'
    }

    environment {
        // Nom de ton image Docker
        IMAGE_NAME = "yappa01/student-app"
        // Clé du projet SonarQube (doit correspondre à ce que tu as mis dans le dashboard Sonar)
        SONAR_PROJECT_KEY = "student-management"
        SONAR_PROJECT_NAME = "Student App"
    }

    stages {
        stage('Checkout') {
            steps {
                // Récupération du code source
                git branch: 'main', url: 'https://github.com/djappa001/JNKS.git'
            }
        }

        stage('Build, Test & Analyze') {
            steps {
                script {
                    // Utilisation de l'environnement SonarQube configuré dans Jenkins
                    withSonarQubeEnv('SonarQube') { 
                        // CRUCIAL : On lance 'clean verify sonar:sonar' en UNE seule commande.
                        // 'verify' génère le rapport JaCoCo (target/site/jacoco/jacoco.xml)
                        // 'sonar:sonar' le lit immédiatement après.
                        sh "mvn clean verify sonar:sonar -Dsonar.projectKey=${SONAR_PROJECT_KEY} -Dsonar.projectName='${SONAR_PROJECT_NAME}'"
                    }
                }
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
                // Génère le fichier .jar dans le dossier target/
                // On saute les tests ici car ils ont déjà été faits à l'étape "Build & Analyze"
                sh 'mvn package -DskipTests' 
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    echo "🔨 Construction de l'image Docker..."
                    // Construction avec le tag du numéro de build (ex: :21)
                    sh "docker build -t ${IMAGE_NAME}:${env.BUILD_NUMBER} ."
                    
                    // Création du tag 'latest' pour la version la plus récente
                    sh "docker tag ${IMAGE_NAME}:${env.BUILD_NUMBER} ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    // Récupération sécurisée des identifiants DockerHub
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-id', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        echo "📤 Envoi vers Docker Hub..."
                        
                        // 1. Connexion
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        
                        // 2. Envoi de la version spécifique
                        sh "docker push ${IMAGE_NAME}:${env.BUILD_NUMBER}"
                        
                        // 3. Envoi de la version latest
                        sh "docker push ${IMAGE_NAME}:latest"
                    }
                }
            }
        }
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
