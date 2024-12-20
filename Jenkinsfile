
pipeline {
    agent any

    environment {
        MAVEN_HOME = '/var/jenkins_home/workspace/testGen/maven'
        JAVA_HOME = '/var/jenkins_home/workspace/testGen/java'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checkout del codice...'
                checkout scm
            }
        }

        stage('Install Java') {
            steps {
                echo "Installiamo Java..."
                script {
                    // Creiamo una cartella per Java e scarichiamo il JDK 8
                    sh """
                        mkdir -p ${JAVA_HOME}
                        curl -sL https://download.java.net/java/GA/jdk8/8u292-b10/5d4f34ff1c214ea8a34b09f13f3253c0/installer/jdk-8u292-linux-x64.tar.gz -o /tmp/jdk.tar.gz
                        tar xz -C ${JAVA_HOME} -f /tmp/jdk.tar.gz --strip-components=1
                        export JAVA_HOME=${JAVA_HOME}
                        export PATH=\$JAVA_HOME/bin:\$PATH
                    """
                }
            }
        }

        stage('Install Maven') {
            steps {
                echo "Installiamo Maven..."
                script {
                    // Creiamo una cartella per Maven e scarichiamo la versione 3.8.6
                    sh """
                        mkdir -p ${MAVEN_HOME}
                        curl -sL https://downloads.apache.org/maven/maven-3/3.8.6/binaries/apache-maven-3.8.6-bin.tar.gz -o /tmp/apache-maven-3.8.6-bin.tar.gz
                        tar xz -C ${MAVEN_HOME} -f /tmp/apache-maven-3.8.6-bin.tar.gz --strip-components=1
                        export MAVEN_HOME=${MAVEN_HOME}
                        export PATH=\$MAVEN_HOME/bin:\$PATH
                    """
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Eseguiamo la build con Maven...'
                script {
                    // Eseguiamo la build con Maven
                    sh """
                        mvn clean install
                    """
                }
            }
        }

        stage('Test') {
            steps {
                echo 'Eseguiamo i test...'
                script {
                    // Eseguiamo i test di copertura con Maven e JaCoCo
                    sh """
                        mvn test
                    """
                }
            }
        }

        stage('Check Test Coverage') {
            steps {
                echo 'Verifica della copertura di test...'
                script {
                    // Estraiamo la percentuale di copertura dei test da JaCoCo
                    def coverage = sh(script: "grep -oP 'Total.*\\K\\d+(\\.\\d+)?' target/site/jacoco/index.html", returnStdout: true).trim()
                    echo "Copertura dei test: ${coverage}%"

                    // Se la copertura è inferiore all'80%, segnala e genera i test con EvoSuite
                    if (coverage.toFloat() < 80.0) {
                        echo "La copertura dei test è inferiore al 80%. Generando i test con EvoSuite..."
                        sh """
                            mvn dependency:copy -Dartifact=org.evosuite:evosuite-standalone-runtime:1.0.4
                            java -cp target/classes:evosuite-standalone-runtime-1.0.4.jar org.evosuite.EvoSuite -class hello.HelloWorld
                            mvn test
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'La pipeline è stata completata con successo!'
        }
        failure {
            echo 'La pipeline è fallita. Verifica i log per ulteriori dettagli.'
        }
    }
}
