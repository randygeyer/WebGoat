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
    
    stage('DEPLOY QA') {
      steps {
        sshagent(['prod']) {
          sh 'ssh -o StrictHostKeyChecking=no ubuntu@34.210.33.150 "sudo bash /prod/apache-tomcat-8.5.45/bin/shutdown.sh"'
          sh 'scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/cardiff-secure-cicd-pipeline/webgoat-container/target/*.war ubuntu@34.210.33.150:/prod/apache-tomcat-8.5.45/webapps/webapp.war'
          sh 'ssh -o StrictHostKeyChecking=no ubuntu@34.210.33.150 "sudo bash /prod/apache-tomcat-8.5.45/bin/startup.sh"'
        }
      }
    }

    stage('DAST ZAP') { 
      steps {
	sh 'docker run -t owasp/zap2docker-stable zap-baseline.py -t http://34.210.33.150:8080/webapp/ || true'
      }
    }
	  
    stage('DAST NIKTO') { 
      steps {
	sh 'docker run --user $(id -u):$(id -g) --rm -v $(pwd):/report -i secfigo/nikto:latest -h 34.210.33.150 -port 8080 -output /report/nikto-output.xml'
      }
    }
	  
    stage('DAST SSLSCAN') { 
      steps {
	sh 'sslyze --regular 34.210.33.150 --json_out sslyze-output.json'
      }
    }
	  
    stage('DAST NMAP') { 
      steps {
	sh 'nmap 34.210.33.150 -oX nmap.xml'
      }
    }

  }
  
}
