pipeline {
    agent any
    
    environment {
        MAVEN_HOME = '/usr/local/maven'  // Sostituire con il percorso Maven corretto
        JAVA_HOME = '/usr/lib/jvm/java-8-openjdk-amd64'  // Sostituire con il percorso Java corretto
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                script {
                    // Esegui il comando Maven per compilare il progetto e eseguire i test
                    sh "'${MAVEN_HOME}/bin/mvn' clean install"
                }
            }
        }
        
        stage('Check Test Coverage') {
            steps {
                script {
                    // Analizza il report di JaCoCo per la percentuale di copertura
                    def coverage = sh(script: "'${MAVEN_HOME}/bin/mvn' jacoco:report", returnStdout: true).trim()
                    def coveragePercentage = sh(script: "grep -oP 'Total.*\K\d+(\.\d+)?' target/site/jacoco/index.html", returnStdout: true).trim()
                    
                    echo "Coverage Percentage: ${coveragePercentage}%"

                    // Evidenzia la copertura in grassetto se inferiore all'80%
                    if (coveragePercentage.toFloat() < 80) {
                        echo "\033[1m**Coverage is below 80%! (${coveragePercentage}%)**\033[0m"
                        currentBuild.result = 'UNSTABLE'
                    } else {
                        echo "\033[1mCoverage is above 80% (${coveragePercentage}%)\033[0m"
                    }
                }
            }
        }
        
        stage('Generate Missing Tests') {
            when {
                expression {
                    // Verifica se la copertura è inferiore all'80%
                    def coveragePercentage = sh(script: "grep -oP 'Total.*\K\d+(\.\d+)?' target/site/jacoco/index.html", returnStdout: true).trim()
                    return coveragePercentage.toFloat() < 80
                }
            }
            steps {
                script {
                    echo "Generating missing tests to improve coverage..."
                    
                    // Esegui EvoSuite per generare i test mancanti
                    sh "'${MAVEN_HOME}/bin/mvn' clean test -Devosuite"
                }
            }
        }

        stage('Rebuild and Test with New Tests') {
            when {
                expression {
                    // Assicura che il passo 'Generate Missing Tests' sia stato eseguito
                    def coveragePercentage = sh(script: "grep -oP 'Total.*\K\d+(\.\d+)?' target/site/jacoco/index.html", returnStdout: true).trim()
                    return coveragePercentage.toFloat() < 80
                }
            }
            steps {
                script {
                    // Ricompila il progetto con i nuovi test generati
                    sh "'${MAVEN_HOME}/bin/mvn' clean install"
                }
            }
        }

        stage('Final Coverage Check') {
            steps {
                script {
                    // Analizza nuovamente la copertura finale
                    def coveragePercentage = sh(script: "grep -oP 'Total.*\K\d+(\.\d+)?' target/site/jacoco/index.html", returnStdout: true).trim()

                    echo "Final Coverage: ${coveragePercentage}%"
                    if (coveragePercentage.toFloat() < 80) {
                        error "Coverage is still below 80%. Please fix the missing tests."
                    } else {
                        echo "\033[1mFinal Coverage is above 80% (${coveragePercentage}%)\033[0m"
                    }
                }
            }
        }
    }
    
    post {
        always {
            // Rimuove i report di JaCoCo se esistono
            cleanWs()
        }
    }
}

