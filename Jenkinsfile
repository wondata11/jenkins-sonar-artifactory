pipeline {
    agent any
    environment {
        ARTIFACTORY_API_TOKEN = credentials('artifactory-access-token')
        JFROG_PASSWORD = credentials('jfrog-password')
        SONAR_LOGIN_TOKEN = credentials('sonarqube-login-token')
        SONAR_PASSWORD = credentials('sonarqube-password')
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('SonarScanner') {
                    sh "mvn sonar:sonar \
                        -Dsonar.host.url=http://lab.cloudsheger.com:9000 \
                        -Dsonar.projectKey=CloudSheger-Java-app \
                        -Dsonar.login=${SONAR_LOGIN_TOKEN} \
                        -Dsonar.password=${SONAR_PASSWORD} \
                        -Dsonar.projectName='CloudSheger-Java-app' \
                        -Dsonar.projectVersion=1.0.0 \
                        -Dsonar.java.binaries=target/classes"
                }
            }
        }
    }
    post{
        failure{
            slackSend( channel: "#build-status", token: "slack-jenkins-secret-key token", color: "good", message: "${custom_msg()}")
        }
     }
}


def custom_msg()
{
  def JENKINS_URL= "http://lab.cloudsheger.com:8080"
  def JOB_NAME = env.JOB_NAME
  def BUILD_ID= env.BUILD_ID
  def JENKINS_LOG= " FAILED: Job [${env.JOB_NAME}] Logs path: ${JENKINS_URL}/job/${JOB_NAME}/${BUILD_ID}/consoleText"
  return JENKINS_LOG
}