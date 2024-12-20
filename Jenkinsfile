pipeline {
    agent any

    environment {
        MAVEN_VERSION = '3.8.6'  // Imposta la versione di Maven che desideri utilizzare
    }

    stages {
        stage('Install Maven') {
            steps {
                script {
                    // Scarica e installa Maven a runtime
                    echo "Installing Maven..."
                    sh """
                        curl -s https://archive.apache.org/dist/maven/maven-3/3.8.6/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz | tar xz -C /opt
                        export MAVEN_HOME=/opt/apache-maven-${MAVEN_VERSION}
                        export PATH=\$MAVEN_HOME/bin:\$PATH
                    """
                }
            }
        }
        
        stage('Build and Test') {
            steps {
                script {
                    // Esegui i test con Maven
                    echo "Running tests with Maven..."
                    sh 'mvn clean test'
                }
            }
        }

        stage('JUnit Results') {
            steps {
                // Raccogli i risultati dei test con JUnit (assumendo che i file di report siano nella cartella target)
                echo "Collecting JUnit test results..."
                junit '**/target/test-*.xml'  // Aggiorna il pattern del file se i tuoi risultati sono in un'altra cartella
            }
        }
    }

    post {
        always {
            // Esegui operazioni post-build (opzionale, come la pulizia)
            echo "Post-build actions..."
        }

        success {
            // Azioni da eseguire se il build ha successo
            echo "Build and tests successful!"
        }

        failure {
            // Azioni da eseguire se il build fallisce
            echo "Build or tests failed."
        }
    }
}

