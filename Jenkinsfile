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
          sh 'mvn sonar:sonar -Dsonar.projectKey=devsecops -Dsonar.host.url=http://192.168.48.136:9000 -Dsonar.login=d2b2459aca24add985f6d4b71bf11efd9a38a26f'
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