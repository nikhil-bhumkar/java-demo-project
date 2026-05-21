
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

                    PREVIOUS_COMMIT = sh(
                        script: "git rev-parse HEAD~1",
                        returnStdout: true
                    ).trim()

                    CURRENT_COMMIT = sh(
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

        /*
        OPTIONAL SECURITY SCAN

        stage('Fortify Scan') {
            steps {

                sh '''
                echo "Starting Fortify Scan..."

                sourceanalyzer -b java-demo-project -clean

                sourceanalyzer -b java-demo-project src/

                sourceanalyzer -b java-demo-project \
                -scan -f java-demo-project.fpr

                echo "Fortify Scan Completed"
                '''
            }
        }
        */

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

        /*
        OPTIONAL NEXUS UPLOAD

        stage('Upload JAR to Nexus') {
            steps {

                withCredentials([usernamePassword(
                    credentialsId: 'nexus-creds',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {

                    sh '''
                    echo "Uploading JAR to Nexus..."

                    curl -v -u $NEXUS_USER:$NEXUS_PASS \
                    --upload-file target/*.jar \
                    http://YOUR_NEXUS_IP:8081/repository/maven-releases/com/demo/java-demo-project/${VERSION}/java-demo-project-${VERSION}.jar
                    '''
                }
            }
        }
        */
    }

    post {

        success {

            script {

                echo 'CI PIPELINE SUCCESS'

                def JIRA_ID = env.BRANCH_NAME.tokenize('-')[0]

                echo "Jira Ticket: ${JIRA_ID}"

                sh """
                curl -X POST \
                -H "Content-Type: application/json" \
                -H "X-Automation-Webhook-Token: 4b68e430d8045467ecf214c0c38b5b104af26d45" \
                --data '{
                  "issues": ["${JIRA_ID}"]
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
