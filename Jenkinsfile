pipeline {
  agent any
  
  tools {
    maven 'Maven'
  }
  
  stages {
      
    stage ('SECRETS') {
      steps {
        sh 'rm trufflehog | true'
        sh 'docker run gesellix/trufflehog --json https://github.com/cmarcond/WebGoat-1.git > trufflehog'
        sh 'cat trufflehog'
      }
    }

    stage ('SAST OWASP') {
      steps {
        sh 'rm owasp | true'
        sh 'wget "https://raw.githubusercontent.com/cmarcond/WebGoat-1/master/owasp-dependency-check.sh"'
        sh 'chmod +x owasp-dependency-check.sh'
        sh 'bash owasp-dependency-check.sh'
      }
    }

    stage ('SAST SONAQUBE') {
      steps {
        withSonarQubeEnv('sonarqube') {
          sh 'mvn sonar:sonar'
          sh 'cat target/sonar/report-task.txt'
        }
      }
    }


    stage('BUILD') {
      steps {
        sh 'mvn clean package'
      }
    }
    
    stage('DEPLOY PROD') {
      steps {
        sshagent(['prod']) {
          sh 'scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/cardiff-secure-cicd-pipeline/webgoat-container/target/*.war ubuntu@34.210.33.150:/prod/apache-tomcat-8.5.45/webapps/webapp.war'
        }
      }
    }
    
  }
  
}
