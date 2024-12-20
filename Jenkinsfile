pipeline {
    agent any

    environment {
        MAVEN_VERSION = '3.8.6'  // Imposta la versione di Maven che desideri utilizzare
        MAVEN_HOME = "/tmp/apache-maven-${MAVEN_VERSION}"  // Imposta una directory scrivibile per Maven
    }

    stages {
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
                }
            }
        }

        stage('Install EvoSuite') {
            steps {
                script {
                    echo "Installing EvoSuite..."
                    // Scarica e installa EvoSuite
                    sh """
                        curl -s https://github.com/EvoSuite/evosuite/releases/download/v1.0.6/evosuite-1.0.6.jar -o /tmp/evosuite-1.0.6.jar
                    """
                }
            }
        }

        stage('Initial JaCoCo Coverage') {
            steps {
                script {
                    echo "Checking initial JaCoCo code coverage..."
                    // Verifica la copertura del codice prima di generare i test
                    sh 'mvn clean test jacoco:report'
                    // Visualizza la copertura del codice prima della generazione dei test
                    jacoco execPattern: '**/target/jacoco.exec', classPattern: '**/target/classes', sourcePattern: '**/src/main/java'
                }
            }
        }

        stage('Generate Tests with EvoSuite') {
            steps {
                script {
                    echo "Generating tests with EvoSuite..."
                    // Genera i test con EvoSuite (specifica la directory dei sorgenti Java)
                    sh """
                        java -jar /tmp/evosuite-1.0.6.jar -class <nome.classe> -projectCP <path-al-classpath>
                    """
                }
            }
        }

        stage('Run Tests, Recalculate Coverage') {
            steps {
                script {
                    echo "Running tests and recalculating code coverage with JaCoCo..."
                    // Esegui Maven per il build, i test e generare la copertura con JaCoCo dopo i test generati
                    sh 'mvn clean test jacoco:report'
                }
            }
        }

        stage('JUnit Results') {
            steps {
                script {
                    echo "Collecting JUnit test results..."
                    // Raccogli i risultati dei test JUnit
                    junit '**/target/test-*.xml'  // Aggiungi il pattern corretto per i file di report JUnit
                }
            }
        }

        stage('Final JaCoCo Coverage') {
            steps {
                script {
                    echo "Collecting final JaCoCo code coverage..."
                    // Raccogli i risultati della copertura del codice JaCoCo dopo l'esecuzione dei test
                    jacoco execPattern: '**/target/jacoco.exec', classPattern: '**/target/classes', sourcePattern: '**/src/main/java'
                }
            }
        }
    }

    post {
        always {
            // Azioni da eseguire sempre dopo il build
            echo "Post-build actions..."
        }

        success {
            // Azioni da eseguire se il build ha successo
            echo "Build, tests, and coverage were successful!"
        }

        failure {
            // Azioni da eseguire se il build fallisce
            echo "Build, tests, or coverage failed."
        }
    }
}
