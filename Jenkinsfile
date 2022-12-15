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
    
    /*...........................Pre-commit hooks............................*/
    stage ('Pre-commit hooks') {
      steps {
        sh '''
         rm -rf .git/hooks/pre-commit | true
         curl https://raw.githubusercontent.com/mmukul/pre-commit-hooks/main/install.sh > install-precommit.sh
         chmod +x install-precommit.sh
         ./install-precommit.sh pre-commit
         '''
      }
    }

    /*...........................Git Secrets................................*/
    stage ('Scan Git Secrets') {
      steps {
        sh 'rm trufflehog || true'
        sh 'docker run gesellix/trufflehog --json https://github.com/mmukul/webapp > reports/trufflehog'
      }
    }
    
    /*....................Software Composition Analysis....................*/
    stage ('Vulnerability Scan - SCA') {
      steps {
         sh 'rm owasp* || true'
         sh 'wget "https://raw.githubusercontent.com/mmukul/webapp/master/owasp-dependency-check.sh" '
         sh 'chmod +x owasp-dependency-check.sh'
         sh 'bash owasp-dependency-check.sh'
         sh 'mv dependency-check-report.xml reports/dependency-check-report.xml'
        }
      }
    
    /*...........................SAST................................*/
    stage ('SonarQube - SAST') {
      steps {
          /*sh 'docker run -d -p 9000:9000 -p 9002:9002 owasp/sonarqube || true'*/
          sh 'mvn sonar:sonar -Dsonar.projectKey=devsecops -Dsonar.host.url=http://localhost:9000 -Dsonar.login=d5dc048c2ed785e40716f6ae1e17e47a1df847a2'
        }
      }
    
    stage ('Build') {
      steps {
      sh 'mvn clean package'
       }
    }

    /*...........................DAST................................*/
    stage ('OWASP ZAP - DAST') {
      steps {
        /*sh 'docker run --name webgoat -p 8080:8080 -p 9090:9090 -d webgoat/goatandwolf'*/
        sh '''
          #IPADD=$(ip -f inet -o addr show ens33 | awk '{print $4}' | cut -d '/' -f 1)
          IPADD=$(docker inspect webgoat | grep IPAddress |grep 172* |head -1 | awk '{ print $2 }' | cut -d '"' -f 2)
          docker run --user $(id -u):$(id -g) -v $(pwd):/zap/wrk/:rw --rm -t owasp/zap2docker-stable zap-baseline.py -t http://${IPADD}:8080/WebGoat > reports/zap-baseline-scan.html || true
          '''
          }
        }
      }
    }