pipeline {
    agent any

    environment {
        MVN_OPTS = '-B -DskipTests'   // batch mode, skip tests for faster build (optional)
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/yourusername/your-java-webapp.git'
            }
        }

        stage('Build') {
            steps {
                sh "mvn clean package ${env.MVN_OPTS}"
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.war', fingerprint: true
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'tomcat-server', 
                                                  usernameVariable: 'TOMCAT_USER', 
                                                  passwordVariable: 'TOMCAT_PASS')]) {
                    sh """
                    mvn tomcat7:redeploy \
                        -Dtomcat.username=$TOMCAT_USER \
                        -Dtomcat.password=$TOMCAT_PASS
                    """
                }
            }
        }

        stage('Deploy to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-credentials', 
                                                  usernameVariable: 'NEXUS_USER', 
                                                  passwordVariable: 'NEXUS_PASS')]) {
                    sh """
                    mvn deploy \
                        -Dnexus.username=$NEXUS_USER \
                        -Dnexus.password=$NEXUS_PASS
                    """
                }
            }
        }
    }

    post {
        failure {
            echo 'Build failed! Check logs for errors.'
        }
        success {
            echo 'Build and deployment successful!'
        }
    }
}
