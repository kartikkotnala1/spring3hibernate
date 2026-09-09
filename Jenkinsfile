pipeline {
    agent any

    parameters {
        booleanParam(name: 'RUN_CODE_QUALITY', defaultValue: true, description: 'Run code quality analysis')
        booleanParam(name: 'RUN_CODE_COVERAGE', defaultValue: true, description: 'Run code coverage analysis')
    }

    stages {

        stage('Code Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/kartikkotnala1/spring3hibernate.git'
            }
        }

        stage('Parallel Checks') {
            parallel {

                stage('Code Stability') {
                    steps {
                        sh 'mvn clean test'
                    }
                    post {
                        always {
                            junit 'target/surefire-reports/*.xml'
                        }
                    }
                }

                stage('Code Quality Analysis') {
                    when {
                        expression { params.RUN_CODE_QUALITY == true }
                    }
                    steps {
                        sh 'mvn checkstyle:checkstyle'
                    }
                }

                stage('Code Coverage Analysis') {
                    when {
                        expression { params.RUN_CODE_COVERAGE == true }
                    }
                    steps {
                        sh 'mvn org.jacoco:jacoco-maven-plugin:0.8.11:prepare-agent test org.jacoco:jacoco-maven-plugin:0.8.11:report'
                    }
                }
            }
        }

        stage('Generate Reports') {
            steps {
                sh 'mkdir -p reports && cp -r target/site/* reports/ || true'
            }
        }

        stage('Approval') {
            steps {
                input message: 'Approve publishing artifacts?', ok: 'Approve'
            }
        }

        stage('Publish Artifacts') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar, reports/**', fingerprint: true
            }
        }
    }

    post {
        success {
            slackSend(channel: 'new-channel', message: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
            mail to: 'kartikotnal05@gmail.com',
                 subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Build succeeded and artifacts published: ${env.BUILD_URL}"
        }
        failure {
            slackSend(channel: 'new-channel', message: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
            mail to: 'kartikotnal05@gmail.com',
                 subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Build failed: ${env.BUILD_URL}"
        }
        aborted {
            mail to: 'kartikotnal05@gmail.com',
                 subject: "DENIED/ABORTED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Approval was denied or build aborted: ${env.BUILD_URL}"
        }
    }
}
