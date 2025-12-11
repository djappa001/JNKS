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
                        sh "mvn clean verify sonar:sonar -Dsonar.projectKey=${SONAR_PROJECT_KEY} -Dsonar.projectName='${SONAR_PROJECT_NAME}'"
                    }
                }
            }
        }
       
        stage('Code Packaging'){
            steps{
                sh 'mvn package -DskipTests' 
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    echo "🔨 Construction de l'image Docker : ${env.BUILD_NUMBER}..."
                    sh "docker build -t ${IMAGE_NAME}:${env.BUILD_NUMBER} ."
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

        // --- ÉTAPE 1 : Kubernetes Deploy (Rapide ~1s) ---
        // Cette étape envoie juste l'ordre de mise à jour au cluster
        stage('Kubernetes Deploy') {
            steps {
                script {
                    echo "🚀 Envoi de la configuration à Kubernetes..."
                    
                    // On s'assure que le namespace existe
                    sh "kubectl create namespace ${K8S_NAMESPACE} || true"
                    
                    // Si tu as des fichiers YAML dans ton git, tu peux décommenter la ligne suivante :
                    // sh "kubectl apply -f k8s/ -n ${K8S_NAMESPACE}"

                    // Mise à jour de l'image pour utiliser la nouvelle version buildée
                    sh "kubectl set image deployment/${K8S_DEPLOYMENT_NAME} spring-app=${IMAGE_NAME}:${env.BUILD_NUMBER} -n ${K8S_NAMESPACE}"
                }
            }
        }

        // --- ÉTAPE 2 : Deploy MySQL & Spring Boot on K8s (Validation ~40s) ---
        // Cette étape attend que les pods soient réellement en vert (Running)
        stage('Deploy MySQL & Spring Boot on K8s') {
            steps {
                script {
                    echo "⏳ Attente du déploiement correct..."
                    
                    // On attend que le déploiement soit terminé avec succès
                    sh "kubectl rollout status deployment/${K8S_DEPLOYMENT_NAME} -n ${K8S_NAMESPACE}"
                    
                    // (Optionnel) Vérification rapide des pods
                    sh "kubectl get pods -n ${K8S_NAMESPACE}"
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
