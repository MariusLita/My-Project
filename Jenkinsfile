pipeline {
    agent any
    tools {
        maven "MAVEN3.9"
        jdk "JDK17"
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Fetch code') {
            steps{
                git branch: 'main', url: 'https://github.com/MariusLita/My-Project.git'
            }
        }



        stage('Build') {
            steps{
                sh 'mvn install -DskipTests'
            }
            post {
                success {
                    echo 'Archiving artifact'
                    archiveArtifacts artifacts: '**/*.jar'
                }
                failure {
                    echo 'Build Failed'
                }
            }
        }

        stage('Unit Test') {
            steps{
                sh 'mvn test'
            }
        }

        stage('Checkstyle Analysis') {
            steps{
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage("Sonar Code Analysis") {
            environment {
                scannerHome = tool 'sonar6.2'
            }
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh '''${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=application \
                        -Dsonar.projectName=application \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/test-classes/com/example/phonebook/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports \
                        -Dsonar.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Upload Artifact"){
            steps{
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '172.17.0.6:8081',
                    groupId: 'QA',
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: 'app-repo',
                    credentialsId: 'nexuslogin',
                    artifacts: [
                        [artifactId: 'app-artifact', 
                        classifier: '',
                        file: 'target/phonebook-0.0.1-SNAPSHOT.jar',
                        type: 'jar']
                    ]
                )
            }
        }

    }
}
