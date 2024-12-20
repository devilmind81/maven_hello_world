pipeline {
    agent any

    environment {
        // Variabili di ambiente per Maven
        MAVEN_VERSION = '3.8.6'
        MAVEN_HOME = "/tmp/apache-maven-${MAVEN_VERSION}"
        PATH = "$MAVEN_HOME/bin:$PATH"
        
        // Variabile per il percorso di EvoSuite
        EVOSUITE_PATH = "/tmp/evosuite-1.0.6.jar"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                echo "Checking out the repository..."
                checkout scm
            }
        }

        stage('Install Maven') {
            steps {
                script {
                    echo "Installing Maven..."
                    // Installa Maven in una directory temporanea
                    sh """
                        curl -s https://archive.apache.org/dist/maven/maven-3/3.8.6/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz | tar xz -C /tmp
                        export MAVEN_HOME=/tmp/apache-maven-${MAVEN_VERSION}
                        export PATH=\$MAVEN_HOME/bin:\$PATH
                    """
                    // Verifica che Maven sia stato installato correttamente
                    sh 'echo $PATH'
                    sh 'mvn -version'
                }
            }
        }

        stage('Install EvoSuite') {
            steps {
                script {
                    echo "Installing EvoSuite..."
                    // Scarica EvoSuite nella directory temporanea
                    sh """
                        curl -s https://github.com/EvoSuite/evosuite/releases/download/v1.0.6/evosuite-1.0.6.jar -o $EVOSUITE_PATH
                    """
                }
            }
        }

        stage('Initial JaCoCo Coverage') {
            steps {
                echo "Checking initial JaCoCo code coverage..."
                // Esegui i test per ottenere la copertura iniziale
                sh """
                    mvn clean test jacoco:report
                """
                // Visualizza il rapporto di copertura JaCoCo
                sh """
                    tail -n 10 target/site/jacoco/index.html
                """
            }
        }

        stage('Generate Tests with EvoSuite') {
            steps {
                echo "Generating tests using EvoSuite..."
                // Genera i test con EvoSuite per tutte le classi nel package principale
                sh """
                    java -jar $EVOSUITE_PATH -class com.yourpackage.YourMainClass -projectDir target/evoTests
                """
            }
        }

        stage('Run Tests, Recalculate Coverage') {
            steps {
                echo "Running tests and recalculating coverage..."
                // Esegui di nuovo i test con la generazione di nuovi test da EvoSuite e calcola la copertura
                sh """
                    mvn clean test jacoco:report
                """
                // Visualizza il rapporto di copertura JaCoCo
                sh """
                    tail -n 10 target/site/jacoco/index.html
                """
            }
        }

        stage('JUnit Results') {
            steps {
                echo "Collecting JUnit results..."
                // Raccogli e pubblica i risultati dei test JUnit
                junit '**/target/test-*.xml'
            }
        }

        stage('Final JaCoCo Coverage') {
            steps {
                echo "Checking final JaCoCo coverage..."
                // Verifica la copertura finale di JaCoCo dopo aver eseguito i nuovi test
                sh """
                    tail -n 10 target/site/jacoco/index.html
                """
            }
        }

        stage('Post-build Actions') {
            steps {
                echo "Post-build actions..."
                // Stampa un messaggio in base al risultato della build
                script {
                    def buildStatus = currentBuild.currentResult
                    if (buildStatus == 'SUCCESS') {
                        echo "Build, tests, and coverage calculations were successful!"
                    } else {
                        echo "Build, tests, or coverage failed."
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up after the build..."
            // Pulizia finale, se necessario
            deleteDir()
        }
        success {
            echo "The pipeline completed successfully!"
        }
        failure {
            echo "The pipeline failed. Please check the logs for details."
        }
    }
}

