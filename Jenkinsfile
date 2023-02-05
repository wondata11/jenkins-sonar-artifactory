pipeline {
    agent any
    environment {
        ARTIFACTORY_API_TOKEN = credentials('jboss-access-token')
        JFROG_PASSWORD = credentials('jfrog-password')
        SONAR_LOGIN_TOKEN = credentials('sonarqube-login-token')
        SONAR_PASSWORD = credentials('sonarqube-password')
    }

    stages {
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
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
                        -Dsonar.projectKey=demo \
                        -Dsonar.login=${SONAR_LOGIN_TOKEN} \
                        -Dsonar.projectName='demo' \
                        -Dsonar.projectVersion=1.0.0 \
                        -Dsonar.java.binaries=target/classes"
                }
            }
        }
        stage('Push to Artifactory') {
            steps {
                withCredentials([string(credentialsId: 'artifactory-api-token', variable: 'ARTIFACTORY_API_TOKEN')]) {
                    sh 'curl -u ${ARTIFACTORY_API_TOKEN} -X PUT "http://lab.cloudsheger.com:8082/artifactory/java-web-app/demo/${env.BUILD_NUMBER}/demo-${env.BUILD_NUMBER}.jar" -T target/demo-0.0.1*.jar'
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

/*
stage('Deploy') {
            steps {
                withMaven(maven: 'Maven 3') {
                    sh "mvn deploy -Dmaven.test.skip=true -DaltDeploymentRepository=snapshot-repo::default::http://lab.cloudsheger.com:8082/artifactory/java-web-app/"
                }
            }
        }
*/        

