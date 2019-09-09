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

    stage ('secrets') {
      steps {
        sh 'rm trufflehog | true'
        sh 'docker run gesellix/trufflehog --json https://github.com/cmarcond/WebGoat-1.git > trufflehog'
        sh 'cat trufflehog'
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
