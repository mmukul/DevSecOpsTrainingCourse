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

    stage ('Clean Images') {
     steps {
        sh '''
            docker rm -vf $(docker ps -aq)
            docker rmi -f $(docker images -aq)
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
    
    stage ('SCA - Dependency Check') {
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
          sh 'docker rm -f owasp/sonarqube || true'
          sh 'docker rmi -f owasp/sonarqube || true'
          sh 'docker run -d -p 9095:9095 -p 9090:9090 owasp/sonarqube'
          sh 'mvn sonar:sonar -Dsonar.projectKey=webapp -Dsonar.host.url=http://localhost:9095 -Dsonar.login=d9347a83d86b83ba5da564459f8d115a5f42106a'
        }
      }
    
    stage ('Build') {
      steps {
      sh 'mvn clean package'
       }
    }

    stage ('DAST') {
      steps {
        sshagent(['zap']) {
         sh 'docker run -t owasp/zap2docker-stable zap-baseline.py -t http://localhost:8080/webapp/" || true'
        }
      }
    }
  }
}