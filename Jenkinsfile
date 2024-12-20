pipeline {
    agent any
    environment {
        MAVEN_VERSION = '3.8.1'
        JAVA_VERSION = '17'
        MAVEN_HOME = tool name: 'Maven', type: 'Tool'  // Usa Maven configurato in Jenkins
        JAVA_HOME = tool name: 'JDK 17', type: 'Tool' // Usa JDK configurato in Jenkins
        PATH = "${MAVEN_HOME}/bin:${JAVA_HOME}/bin:${env.PATH}"
    }
    stages {
        stage('Install Tools and Dependencies') {
            steps {
                script {
                    // Installa Maven a runtime
                    sh '''
                    echo "Installing Maven..."
                    wget https://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz
                    tar xzf apache-maven-${MAVEN_VERSION}-bin.tar.gz
                    export MAVEN_HOME=$(pwd)/apache-maven-${MAVEN_VERSION}
                    export PATH=$MAVEN_HOME/bin:$PATH
                    echo "Maven installed"
                    '''

                    // Installa Java a runtime (se necessario)
                    sh '''
                    echo "Installing Java 17..."
                    wget https://download.java.net/java/GA/jdk${JAVA_VERSION}/binaries/openjdk-${JAVA_VERSION}_linux-x64_bin.tar.gz
                    tar xzf openjdk-${JAVA_VERSION}_linux-x64_bin.tar.gz
                    export JAVA_HOME=$(pwd)/jdk-${JAVA_VERSION}
                    export PATH=$JAVA_HOME/bin:$PATH
                    echo "Java 17 installed"
                    '''
                    
                    // Installa JaCoCo (se necessario)
                    sh '''
                    echo "Installing JaCoCo..."
                    wget https://repo1.maven.org/maven2/org/jacoco/org.jacoco.core/0.8.7/org.jacoco.core-0.8.7.jar
                    echo "JaCoCo installed"
                    '''

                    // Installa EvoSuite (se necessario)
                    sh '''
                    echo "Installing EvoSuite..."
                    wget https://repo1.maven.org/maven2/org/evosuite/evosuite/1.1.0/evosuite-1.1.0.jar
                    echo "EvoSuite installed"
                    '''
                }
            }
        }

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                sh 'mvn clean install'  // Costruisci il progetto con Maven
            }
        }

        stage('Code Coverage with JaCoCo') {
            steps {
                sh 'mvn jacoco:report'  // Genera il report di coverage con JaCoCo
            }
        }

        stage('Generate Tests with EvoSuite') {
            steps {
                sh 'java -jar evosuite-1.1.0.jar -class com.example.MyClass'  // Usa EvoSuite per generare test
            }
        }

        stage('Run Generated Tests') {
            steps {
                sh 'mvn test'  // Esegui i test generati
            }
        }
    }

    post {
        always {
            junit '**/target/surefire-reports/*.xml'  // Mostra i risultati dei test
        }
    }
}

