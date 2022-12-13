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
          sh 'mvn sonar:sonar -Dsonar.projectKey=devsecops -Dsonar.host.url=http://localhost:9000 -Dsonar.login=1011d3cc19b970778e3986418a422f7dca5d827f'
        }
      }
    
    stage ('Build') {
      steps {
      sh 'mvn clean package'
       }
    }

    stage ('OWASP ZAP - DAST') {
      steps {
        sh 'docker network create zapnet'
        sh 'docker run --name goatandwolf -p 8081:8081 -p 9090:9090 -d --net zapnet webgoat/goatandwolf'
        sh 'docker run --net zapnet -v $(pwd):/zap/wrk/:rw -t owasp/zap2docker-stable zap-full-scan.py -I -j -m 10 -T 60 -t http://172.18.0.2:8081/WebGoat -r zap-full-scan-without-user.html'
      }
    }
  }
}

post {
  always {
    dependencyCheckPublisher pattern: 'dependency-check-report.xml'
  }
}