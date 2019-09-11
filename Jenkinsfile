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

    stage('COMMIT APPROVAL'){
      steps {
         input "Deploy to QA?"
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
	sh 'docker run --rm -t nablac0d3/sslyze --regular www.github.com:443'
      }
    }
	  
    stage('DAST NMAP') { 
      steps {
	sh 'nmap 34.210.33.150 -oX nmap.xml'
      }
    }

    stage('HARDENING COMPLIANCE INSPEC') { 
      steps {
	sh 'inspec exec https://github.com/dev-sec/linux-baseline -t ssh://ubuntu@34.210.33.150 -i /tmp/pipeline.pem --chef-license=accept || true'
      }
    }

    stage('HARDENING COMPLIANCE OPENSCAP') {
      steps {
        sshagent(['prod']) {
          sh 'ssh -o StrictHostKeyChecking=no ubuntu@34.210.33.150 "wget -P /tmp/ https://people.canonical.com/~ubuntu-security/oval/com.ubuntu.bionic.cve.oval.xml"'
	  sh 'ssh -o StrictHostKeyChecking=no ubuntu@34.210.33.150 "oscap oval eval --results /tmp/oscap_results.xml --report /tmp/oscap_report.html /tmp/com.ubuntu.bionic.cve.oval.xml"'
          sh 'scp -o StrictHostKeyChecking=no ubuntu@34.210.33.150:/tmp/oscap_report.html oscap_results.html'
	  sh 'sh 'cat oscap_results.html'
        }
      }
    }

    stage('HARDENING QA PATCH') { 
      steps {
	sh 'echo "[prod]" > inventory.ini'
	sh 'echo "34.210.33.150 ansible_user=ubuntu  ansible_ssh_private_key_file=/tmp/pipeline.pem" >> inventory.ini'
	sh 'ansible-galaxy install dev-sec.os-hardening'
	sh 'ansible-playbook -i inventory.ini ansible-hardening.yml'
      }
    }  

    stage('QA APPROVAL'){
      steps {
         input "Deploy to Prod?"
      }
    }
	  
    stage('DEPLOY PROD') {
      steps {
        sshagent(['prod']) {
          sh 'ssh -o StrictHostKeyChecking=no ubuntu@34.210.33.150 "sudo bash /prod/apache-tomcat-8.5.45/bin/shutdown.sh"'
          sh 'scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/cardiff-secure-cicd-pipeline/webgoat-container/target/*.war ubuntu@34.210.33.150:/prod/apache-tomcat-8.5.45/webapps/webapp.war'
          sh 'ssh -o StrictHostKeyChecking=no ubuntu@34.210.33.150 "sudo bash /prod/apache-tomcat-8.5.45/bin/startup.sh"'
        }
      }
    }

  }
  
}
