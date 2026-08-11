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
        APP_VERSION = '1.0-SNAPSHOT'
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
                sh 'git config --global --add safe.directory "$WORKSPACE"'
                sh 'git log --oneline -5'
            }
        }

        stage('Build') {
            steps {
                sh "${MVN_CMD} clean compile"
            }
        }

        stage('Test') {
            steps {
                sh "${MVN_CMD} test"
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Quality Analysis') {
            tools { maven 'Maven-3.9' }
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                        mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                          -Dsonar.projectKey=${env.APP_NAME} \
                          -Dsonar.projectName="TechBuild ${env.APP_NAME}" \
                          -Dsonar.projectVersion=${env.APP_VERSION} \
                          -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml \
                          -B
                    """
                }
            }
        }

        stage('Quality Gate') {
            agent none
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package & Archive') {
            tools { maven 'Maven-3.9' }
            steps {
                sh "mvn package -DskipTests -B -Drevision=${env.APP_VERSION}"
                sh 'ls -lh target/'
                archiveArtifacts(
                    artifacts: 'target/*.war',
                    fingerprint: true
                )
            }
        }

        stage('Publish Artifact') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '98.91.250.253:8081',
                    groupId: 'io.techbuild',
                    version: env.APP_VERSION,
                    repository: 'techbuild-releases',
                    credentialsId: 'nexus-creds',
                    artifacts: [[
                        artifactId: env.APP_NAME,
                        classifier: '',
                        file: "target/${env.APP_NAME}-${env.APP_VERSION}.war",
                        type: 'war'
                    ]]
                )
            }
        }
    }

    post {
        success {
            echo "PIPELINE SUCCESS — ${env.APP_NAME} v${env.APP_VERSION}"
        }

        failure {
            echo "PIPELINE FAILED — check logs at ${env.BUILD_URL}"
        }

        always {
            cleanWs()
        }
    }
}