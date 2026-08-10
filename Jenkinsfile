pipeline {
    agent {
        docker {
            image 'maven:3.9.6-eclipse-temurin-17'
            args '-u root:root -v $HOME/.m2:/root/.m2'
            reuseNode true
        }
    }

    environment {
        MAVEN_OPTS = '-Dmaven.repo.local=/root/.m2/repository'
        MVN_CMD = 'mvn -B -ntp'
        APP_NAME = 'hello-world'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm

                echo "Branch: ${env.GIT_BRANCH}"
                echo "Commit: ${env.GIT_COMMIT}"

                sh 'java -version'
                sh 'mvn -version'
                sh 'git log --oneline -5'
            }
        }
    }
}