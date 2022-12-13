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
          sh 'mvn sonar:sonar -Dsonar.projectKey=devsecops -Dsonar.host.url=http://"$(ip -f inet -o addr show ens33 | awk '{print $4}' | cut -d '/' -f 1)":9000 -Dsonar.login=1011d3cc19b970778e3986418a422f7dca5d827f'
        }
      }
    
    stage ('Build') {
      steps {
      sh 'mvn clean package'
       }
    }

    stage ('OWASP ZAP - DAST') {
      steps {
        sh 'docker run -t owasp/zap2docker-weekly zap-baseline.py -t http://"$(ip -f inet -o addr show docker0 | awk '{print $4}' | cut -d '/' -f 1)":10080'
      }
    }
  }
}