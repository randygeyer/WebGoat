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
        sh 'mvn tomcat:run-war clean package'
      }
    }
    
  }
  
}
