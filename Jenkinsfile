pipeline {
    agent any
    environment {
        ISSUE_KEY = "UAT-1"
        JIRA_URL  = "https://nikhildevops.atlassian.net"
        JIRA_USER = "nikhilbhumkar22@gmail.com"
    }
    stages {
        stage('Checkout GitHub Repo') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/nikhil-bhumkar/java-demo-project.git'
            }
        }
        stage('Notify Jira - Build Started') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'jira-token', variable: 'JIRA_TOKEN')]) {
                        sh """
                            curl -s -X POST \
                              -u "${JIRA_USER}:${JIRA_TOKEN}" \
                              -H "Content-Type: application/json" \
                              -d '{"body": "🚀 Jenkins build started"}' \
                              "${JIRA_URL}/rest/api/2/issue/${ISSUE_KEY}/comment"
                        """
                    }
                }
            }
        }
        stage('Build') {
            steps {
                sh 'MAVEN_OPTS="-Xmx128m" mvn -q clean compile'
            }
        }
        stage('Test') {
            steps {
                sh 'MAVEN_OPTS="-Xmx128m" mvn -q test'
            }
        }
        stage('Package') {
            steps {
                sh 'MAVEN_OPTS="-Xmx128m" mvn -q package'
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                    mkdir -p /mnt/apps
                    cp target/*.jar /mnt/apps/
                '''
            }
        }
        stage('Notify Jira - Success') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'jira-token', variable: 'JIRA_TOKEN')]) {
                        sh """
                            curl -s -X POST \
                              -u "${JIRA_USER}:${JIRA_TOKEN}" \
                              -H "Content-Type: application/json" \
                              -d '{"body": "✅ Build + Deployment successful"}' \
                              "${JIRA_URL}/rest/api/2/issue/${ISSUE_KEY}/comment"
                        """
                    }
                }
            }
        }
    }
    post {
        success {
            echo 'Application deployed successfully'
        }
        failure {
            script {
                withCredentials([string(credentialsId: 'jira-token', variable: 'JIRA_TOKEN')]) {
                    sh """
                        curl -s -X POST \
                          -u "${JIRA_USER}:${JIRA_TOKEN}" \
                          -H "Content-Type: application/json" \
                          -d '{"body": "❌ Jenkins pipeline failed"}' \
                          "${JIRA_URL}/rest/api/2/issue/${ISSUE_KEY}/comment"
                    """
                }
            }
            echo 'Pipeline failed!'
        }
    }
}
