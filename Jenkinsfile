pipeline {
    agent any

    tools {
        maven 'M2_HOME' 
        jdk 'JAVA_HOME'
    }

    environment {
        // Ton image Docker Hub
        IMAGE_NAME = "yappa01/student-app"
        // Infos SonarQube
        SONAR_PROJECT_KEY = "student-management"
        SONAR_PROJECT_NAME = "Student App"
        // Namespace Kubernetes
        K8S_NAMESPACE = "devops"
        // Nom du déploiement K8s (défini dans ton spring-deployment.yaml)
        K8S_DEPLOYMENT_NAME = "spring-app"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/djappa001/JNKS.git'
            }
        }

        stage('Build, Test & Analyze') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') { 
                        // On fait tout en une fois : Clean, Compile, Test, et Analyse Sonar
                        sh "mvn clean verify sonar:sonar -Dsonar.projectKey=${SONAR_PROJECT_KEY} -Dsonar.projectName='${SONAR_PROJECT_NAME}'"
                    }
                }
            }
        }
       
        stage('Code Packaging'){
            steps{
                // Création du .jar final sans relancer les tests (déjà faits avant)
                sh 'mvn package -DskipTests' 
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    echo "🔨 Construction de l'image Docker : ${env.BUILD_NUMBER}..."
                    // Construction avec le numéro de build unique
                    sh "docker build -t ${IMAGE_NAME}:${env.BUILD_NUMBER} ."
                    // Tag 'latest' pour la référence
                    sh "docker tag ${IMAGE_NAME}:${env.BUILD_NUMBER} ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-id', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        echo "📤 Envoi vers Docker Hub..."
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push ${IMAGE_NAME}:${env.BUILD_NUMBER}"
                        sh "docker push ${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        // --- NOUVELLE ÉTAPE : DÉPLOIEMENT KUBERNETES ---
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "🚀 Mise à jour du cluster Kubernetes..."
                    
                    // 1. On dit à Kubernetes de changer l'image du déploiement
                    // Il va utiliser la version précise qu'on vient de builder (:23, :24, etc.)
                    // Cela force K8s à télécharger la nouvelle version.
                    sh "kubectl set image deployment/${K8S_DEPLOYMENT_NAME} spring-app=${IMAGE_NAME}:${env.BUILD_NUMBER} -n ${K8S_NAMESPACE}"
                    
                    // 2. On attend que la mise à jour soit terminée pour valider le succès
                    sh "kubectl rollout status deployment/${K8S_DEPLOYMENT_NAME} -n ${K8S_NAMESPACE}"
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline terminé avec SUCCÈS ! Application déployée."
            junit 'target/surefire-reports/*.xml'
        }
        failure {
            echo "❌ Le pipeline a échoué."
        }
    }
}
