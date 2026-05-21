pipeline {
    agent any


    environment {
        VERSION = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout GitHub Repo') {
            steps {
                git branch: 'main',
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
        Fortify SCA Scan Stage

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

        stage('Fortify Report') {
            steps {
                sh '''
                ReportGenerator -format html \
                -f fortify-report.html \
                java-demo-project.fpr
                '''
            }
        }
        */

        stage('Package') {
            steps {
                sh 'MAVEN_OPTS="-Xmx256m" mvn -q package'
            }
        }

        /*
        Nexus Upload + Download

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

        stage('Download JAR from Nexus') {
            steps {
                sh '''
                echo "Downloading JAR from Nexus..."

                mkdir -p /mnt/apps

                curl -L -o /mnt/apps/java-demo-project.jar \
                http://YOUR_NEXUS_IP:8081/repository/maven-releases/com/demo/java-demo-project/${VERSION}/java-demo-project-${VERSION}.jar
                '''
            }
        }
        */

        stage('Deploy Application') {
            steps {
                sh '''
              
                echo "Deploying Application..."
               
                mkdir -p /mnt/apps

                cp target/*.jar /mnt/apps/java-demo-project.jar

                echo "Stopping old application if running..."

                pkill -f java-demo-project.jar || true

                sleep 5

                echo "Starting new application..."

                nohup java -jar /mnt/apps/java-demo-project.jar \
                > /mnt/apps/app.log 2>&1 &

                echo "Application Started Successfully"
                '''
            }
        }

        
    }

    post {

        success {
 
            echo 'Build, Test, Package and Deployment completed successfully.'
        }

        failure {
            echo ' PIPELINE FAILED'
            echo 'Check Jenkins console logs for errors.'
        }

        always {
            echo 'Pipeline execution completed.'

        }
    }
}
