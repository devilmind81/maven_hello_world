pipeline {
    agent any
    environment {
        MAVEN_VERSION = '3.8.1'
        MAVEN_HOME = "${tool 'Maven'}"  // Usa Maven scaricato in automatico
        PATH = "${MAVEN_HOME}/bin:${env.PATH}"
    }
    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Install Maven') {
            steps {
                script {
                    // Scarica Maven a runtime
                    sh '''
                    wget https://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz
                    tar xzf apache-maven-${MAVEN_VERSION}-bin.tar.gz
                    export MAVEN_HOME=$(pwd)/apache-maven-${MAVEN_VERSION}
                    export PATH=$MAVEN_HOME/bin:$PATH
                    '''
                }
            }
        }

        stage('Build and Test') {
            steps {
                sh 'mvn clean test'  // Usa Maven scaricato dinamicamente
            }
        }

        stage('Code Coverage with JaCoCo') {
            steps {
                sh 'mvn jacoco:report'
            }
            post {
                always {
                    publishCoverage adapters: [jacocoAdapter('target/site/jacoco/index.html')]
                }
            }
        }

        stage('Generate Tests with EvoSuite') {
            steps {
                sh 'mvn org.evosuite:evosuite-maven-plugin:generate-tests'
            }
        }

        stage('Run Generated Tests') {
            steps {
                sh 'mvn test'
            }
        }
    }

    post {
        always {
            junit '**/target/surefire-reports/*.xml'
        }
    }
}
