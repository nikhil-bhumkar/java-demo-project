pipeline {
    agent any

    stages {

        stage('Checkout GitHub Repo') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/nikhil-bhumkar/java-demo-project.git'
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
    }

    post {
        success {
            echo 'Application deployed successfully'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
