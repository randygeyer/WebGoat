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
        sh 'rm owasp-dependency-check.sh* || true'
        sh 'wget "https://raw.githubusercontent.com/cmarcond/WebGoat-1/master/owasp-dependency-check.sh"'
        sh 'chmod +x owasp-dependency-check.sh'
        sh 'bash owasp-dependency-check.sh'
	sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.html'
      }
    }
	  
    stage ('SAST SONAQUBE') {
      steps {
        withSonarQubeEnv('sonarqube') {
          sh 'mvn clean sonar:sonar'
          sh 'cat target/sonar/report-task.txt'
        }
      }
    }

    stage('BUILD') {
      steps {
        sh 'mvn clean package'
      }
    }

  }
  
}
