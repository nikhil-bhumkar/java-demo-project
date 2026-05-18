pipeline {
    agent any
    
    triggers {
        GenericTrigger(
            genericVariables: [
                [key: 'ISSUE_KEY', value: '$.issue.key', defaultValue: 'UAT-1'],
                [key: 'PR_ACTION', value: '$.action'],
                [key: 'PR_MERGED', value: '$.pull_request.merged']
            ],
            token: 'github-token',
            causeString: 'Triggered by GitHub PR merge',
            regexpFilterText: '$PR_MERGED',
            regexpFilterExpression: 'true'
        )
    }
    
    environment {
        ISSUE_KEY = "${env.ISSUE_KEY ?: 'UAT-1'}"
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
                jiraComment issueKey: "${ISSUE_KEY}",
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
                    mkdir -p /mnt/apps
                    cp target/*.jar /mnt/apps/
                '''
            }
        }
        stage('Notify Jira - Success') {
            steps {
                jiraComment issueKey: "${ISSUE_KEY}",
                            body: "✅ Build + Deployment successful"
            }
        }
    }
    post {
        failure {
            jiraComment issueKey: "${ISSUE_KEY}",
                        body: "❌ Jenkins pipeline failed"
        }
    }
}
