pipeline {

    agent any

    environment {
        VERSION = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout GitHub Repo') {
            steps {

                git branch: "${env.BRANCH_NAME}",
                    credentialsId: 'github-creds',
                    url: 'https://github.com/nikhil-bhumkar/java-demo-project.git'
            }
        }

        stage('Git Change Detection') {
            steps {

                script {

                    def PREVIOUS_COMMIT = sh(
                        script: "git rev-parse HEAD~1",
                        returnStdout: true
                    ).trim()

                    def CURRENT_COMMIT = sh(
                        script: "git rev-parse HEAD",
                        returnStdout: true
                    ).trim()

                    echo "Previous Commit: ${PREVIOUS_COMMIT}"
                    echo "Current Commit : ${CURRENT_COMMIT}"

                    sh """
                        echo "Changed Files Between Commits:"
                        git diff --name-only ${PREVIOUS_COMMIT} ${CURRENT_COMMIT}
                    """
                }
            }
        }

        stage('Clean') {
            steps {
                sh 'mvn clean'
            }
        }

        stage('Build') {
            steps {
                sh 'MAVEN_OPTS="-Xmx256m" mvn -q compile'
            }
        }

        stage('Test') {
            steps {
                sh 'MAVEN_OPTS="-Xmx256m" mvn -q test'
            }
        }

        stage('Package') {
            steps {
                sh 'MAVEN_OPTS="-Xmx256m" mvn -q package'
            }
        }

        stage('Archive Artifact') {
            steps {

                archiveArtifacts artifacts: 'target/*.jar',
                fingerprint: true

                echo "Artifact Archived Successfully"
            }
        }
    }

    post {

        success {

            script {

                echo 'CI PIPELINE SUCCESS'

                def parts = env.BRANCH_NAME.tokenize('-')
                def JIRA_ID = "${parts[0]}-${parts[1]}"

                echo "Jira Ticket: ${JIRA_ID}"

                sh """
                curl -X POST \
                -H "Content-Type: application/json" \
                -H "X-Automation-Webhook-Token: 4b68e430d8045467ecf214c0c38b5b104af26d45" \
                --data '{
                  "issueKey": "${JIRA_ID}"
                }' \
                "https://api-private.atlassian.com/automation/webhooks/jira/a/57f62d85-1b6e-4a44-af5e-0927141b6746/019e4453-d451-7d8f-9462-0dfdd31b50c4"
                """
            }
        }

        failure {

            echo 'CI PIPELINE FAILED'
            echo 'Check Jenkins console logs.'
        }

        always {

            echo 'CI Pipeline execution completed.'
        }
    }
}
