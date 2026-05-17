pipeline {
    agent any

    environment {
        ISSUE_KEY = "UAT-1"
		JIRA_SITE = 'https://nikhildevops.atlassian.net/' 
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

                jiraComment site: "${JIRA_SITE}",
							issueKey: "${ISSUE_KEY}",
                    body: "🚀 Jenkins build started"
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
                cp target/*.jar /mnt/apps/
                '''
            }
        }

        stage('Notify Jira - Success') {
            steps {

                jiraComment site: "${JIRA_SITE}",
							issueKey: "${ISSUE_KEY}",
                    body: "✅ Build + Deployment successful"
            }
        }
    }

    post {

        success {

            echo 'Application deployed successfully'
        }

        failure {

            jiraComment site: "${JIRA_SITE}",
						issueKey: "${ISSUE_KEY}",
                body: "❌ Jenkins pipeline failed"

            echo 'Pipeline failed!'
        }
    }
}	
