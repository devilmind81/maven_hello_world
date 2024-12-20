pipeline {
    agent any
    tools {
        maven 'Maven_3.8.1'  // Configura Maven nel tuo Jenkins
        jdk 'Java_17'        // Configura la versione di Java
    }
    stages {
        // Stage 1: Checkout del codice dal repository
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        // Stage 2: Build e test con Maven
        stage('Build and Test') {
            steps {
                sh 'mvn clean test'
            }
        }

        // Stage 3: Copertura del codice con JaCoCo
        stage('Code Coverage with JaCoCo') {
            steps {
                sh 'mvn jacoco:report'
            }
            post {
                always {
                    // Usa publishCoverage per pubblicare il report di copertura
                    publishCoverage adapters: [jacocoAdapter('target/site/jacoco/index.html')]
                }
            }
        }

        // Stage 4: Esecuzione di EvoSuite per generare test
        stage('Generate Tests with EvoSuite') {
            steps {
                sh 'mvn org.evosuite:evosuite-maven-plugin:generate-tests'
            }
        }

        // Stage 5: Esecuzione dei test generati
        stage('Run Generated Tests') {
            steps {
                sh 'mvn test'
            }
        }
    }

    post {
        always {
            junit '**/target/surefire-reports/*.xml'  // Pubblica i risultati dei test
        }
    }
}

