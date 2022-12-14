pipeline {
  agent any 
  
  stages {
    stage ('Initialize Environment') {
      steps {
        sh '''
            echo "PATH = ${PATH}"
            echo "MAVEN_HOME = ${MAVEN_HOME}"
            ''' 
      }
    }
    
    stage ('Scan Git Secrets') {
      steps {
        sh 'rm trufflehog || true'
        sh 'docker run gesellix/trufflehog --json https://github.com/mmukul/webapp > trufflehog'
        sh 'cat trufflehog'
      }
    }
    
    stage ('Vulnerability Scan - SCA') {
      steps {
         sh 'rm owasp* || true'
         sh 'wget "https://raw.githubusercontent.com/mmukul/webapp/master/owasp-dependency-check.sh" '
         sh 'chmod +x owasp-dependency-check.sh'
         sh 'bash owasp-dependency-check.sh'
         sh 'cat dependency-check-report.xml'
        
      }
    }

    stage ('SonarQube - SAST') {
      steps {
          /*sh 'docker run -d -p 9000:9000 -p 9002:9002 owasp/sonarqube || true'*/
          sh 'mvn sonar:sonar -Dsonar.projectKey=devsecops -Dsonar.host.url=http://localhost:9000 -Dsonar.login=72908a94f4439ddfaf9a8e44405b3999dfbc8755'
        }
      }
    
    stage ('Build') {
      steps {
      sh 'mvn clean package'
       }
    }

    stage ('OWASP ZAP - DAST') {
      steps {
        /*sh 'docker run --name webgoat -p 8080:8080 -p 9090:9090 -d webgoat/goatandwolf'*/
        sh 'docker run -v $(pwd):/zap/wrk/:rw -t owasp/zap2docker-stable zap-baseline.py -I -j -m 10 -T 60 -t http://192.168.48.136:8080/WebGoat -r zap-baseline-scan.html'
        }
      }
  }
}

post {
  always {
    dependencyCheckPublisher pattern: 'dependency-check-report.xml'
  }
}