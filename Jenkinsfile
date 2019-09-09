pipeline {
  agent any
  
  tools {
    maven 'Maven'
  }
  
  stages {
    stage('Environment Variables') {
      steps {
        sh '''
          echo "PATH=${PATH}"
          echo "M2_HOME=${M2_HOME}"
          '''
      }
    } 
    
    stage('Build') {
      steps {
        sh 'mvn clean package'
      }
    }
    
    stage('Deploy Prod') {
      steps {
        sshagent(['prod']) {
          sh 'scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/cardiff-secure-cicd-pipeline/webgoat-container/target/*.war ubuntu@34.210.33.150:/prod/apache-tomcat-8.5.45/webapps/webapp.war'
        }
      }
    }
    
  }
  
}
