pipeline {
    agent any
    
    environment {
        MAVEN_HOME = '/usr/local/maven'  // Percorso Maven corretto (assumiamo che sia già presente)
        JAVA_HOME = '/usr/lib/jvm/java-8-openjdk-amd64'  // Percorso Java corretto (assumiamo che sia già presente)
        PATH = "${MAVEN_HOME}/bin:${JAVA_HOME}/bin:${env.PATH}"  // Aggiungi Maven e Java al PATH
    }

    stages {
        // 1. Checkout del codice sorgente
        stage('Checkout') {
            steps {
                echo "Checkout del codice..."
                checkout scm
            }
        }

        // 2. Build e test iniziale
        stage('Build and Test') {
            steps {
                echo "Eseguiamo il build e i test..."
                script {
                    // Compilazione e esecuzione dei test
                    sh "'${MAVEN_HOME}/bin/mvn' clean install"
                }
            }
        }
        
        // 3. Verifica della copertura dei test
        stage('Check Test Coverage') {
            steps {
                echo "Verifica della copertura dei test..."
                script {
                    // Estrazione della percentuale di copertura dai report JaCoCo
                    def coveragePercentage = sh(script: "grep -oP 'Total.*\\K\\d+(\\.\\d+)?' target/site/jacoco/index.html", returnStdout: true).trim()
                    
                    echo "Percentuale di copertura: ${coveragePercentage}%"

                    // Se la copertura è inferiore all'80%, segnaliamo in grassetto e impostiamo il risultato come UNSTABLE
                    if (coveragePercentage.toFloat() < 80) {
                        echo "\033[1m**La copertura è inferiore all'80%! (${coveragePercentage}%)**\033[0m"
                        currentBuild.result = 'UNSTABLE'  // Risultato instabile
                    } else {
                        echo "\033[1mLa copertura è superiore all'80% (${coveragePercentage}%)\033[0m"
                    }
                }
            }
        }
        
        // 4. Generazione dei test mancanti se la copertura è inferiore all'80%
        stage('Generate Missing Tests') {
            when {
                expression {
                    // Controlla se la copertura è inferiore all'80%
                    def coveragePercentage = sh(script: "grep -oP 'Total.*\\K\\d+(\\.\\d+)?' target/site/jacoco/index.html", returnStdout: true).trim()
                    return coveragePercentage.toFloat() < 80
                }
            }
            steps {
                echo "Generazione dei test mancanti..."
                script {
                    // Esegui EvoSuite per generare test mancanti
                    sh "'${MAVEN_HOME}/bin/mvn' clean test -Devosuite"
                }
            }
        }

        // 5. Ricompilazione e test con i nuovi test generati
        stage('Rebuild and Test with New Tests') {
            when {
                expression {
                    // Assicurati che la generazione dei test sia avvenuta
                    def coveragePercentage = sh(script: "grep -oP 'Total.*\\K\\d+(\\.\\d+)?' target/site/jacoco/index.html", returnStdout: true).trim()
                    return coveragePercentage.toFloat() < 80
                }
            }
            steps {
                echo "Ricompilazione con i nuovi test generati..."
                script {
                    // Esegui nuovamente la build con i test generati
                    sh "'${MAVEN_HOME}/bin/mvn' clean install"
                }
            }
        }

        // 6. Verifica finale della copertura
        stage('Final Coverage Check') {
            steps {
                echo "Verifica finale della copertura..."
                script {
                    // Estrazione della copertura finale dal report JaCoCo
                    def coveragePercentage = sh(script: "grep -oP 'Total.*\\K\\d+(\\.\\d+)?' target/site/jacoco/index.html", returnStdout: true).trim()
                    
                    echo "Copertura finale: ${coveragePercentage}%"

                    // Se la copertura finale è ancora inferiore all'80%, fallisce la build
                    if (coveragePercentage.toFloat() < 80) {
                        error "La copertura è ancora inferiore all'80%. Si prega di correggere i test mancanti."
                    } else {
                        echo "\033[1mLa copertura finale è sopra l'80% (${coveragePercentage}%)\033[0m"
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo "Pulizia ambiente di lavoro..."
            cleanWs()  // Pulisce la workspace al termine del processo
        }
    }
}
