pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo '🔁 Clonage du repository...'
                git branch: 'main', url: 'https://github.com/djappa001/JNKS.git'
            }
        }

        stage('Analyse Fichiers') {
            steps {
                echo '📁 Analyse des fichiers du projet...'
                sh '''
                    echo "=== CONTENU DU REPOSITORY ==="
                    ls -la
                    echo ""
                    echo "=== FICHIERS TROUVÉS ==="
                    find . -type f -name "*.md" -o -name "*.txt" -o -name "Vagrantfile" -o -name "Jenkinsfile"
                    echo ""
                    echo "=== CONTENU DU README ==="
                    [ -f "README.md" ] && head -20 README.md || echo "README.md non trouvé"
                '''
            }
        }

        stage('Validation') {
            steps {
                echo '✅ Validation du projet...'
                sh '''
                    echo "Vérification des fichiers essentiels:"
                    [ -f "README.md" ] && echo "✓ README.md présent" || echo "✗ README.md manquant"
                    [ -f "Vagrantfile" ] && echo "✓ Vagrantfile présent" || echo "✗ Vagrantfile manquant"
                    [ -f "Jenkinsfile" ] && echo "✓ Jenkinsfile présent" || echo "✗ Jenkinsfile manquant"
                    echo "Validation terminée!"
                '''
            }
        }
    }

    post {
        success {
            echo '🎉 Pipeline exécuté avec succès!'
            archiveArtifacts artifacts: '**/*.md, **/*.txt, Jenkinsfile, Vagrantfile', fingerprint: true
        }
        failure {
            echo '❌ Pipeline a échoué!'
        }
    }
}
