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
         sh 'rm -rf .git/hooks/pre-commit || true'
         sh 'curl https://raw.githubusercontent.com/mmukul/pre-commit-hooks/main/install.sh > install-precommit.sh'
         sh 'chmod +x install-precommit.sh'
         sh './install-precommit.sh pre-commit'
      }
    }

    /*...........................Git Secrets................................*/
    stage ('Scan Git Secrets') {
      steps {
        sh 'mkdir reports || true'
        sh 'rm -f reports/trufflehog || true'
        sh 'docker run gesellix/trufflehog --json https://github.com/mmukul/webapp > reports/trufflehog.json'
      }
    }
    
    /*....................Software Composition Analysis....................*/
    stage ('Software Composition Analysis - SCA') {
      steps {
         sh 'rm owasp* || true'
         sh 'wget "https://raw.githubusercontent.com/mmukul/webapp/master/owasp-dependency-check.sh" '
         sh 'chmod +x owasp-dependency-check.sh'
         sh 'bash owasp-dependency-check.sh'
         sh 'mv dependency-check-report.* reports/'
        }
      }
    
    /*...........................SAST................................*/
    stage ('SonarQube - SAST') {
      steps {
          /*sh 'docker run -d -p 9000:9000 -p 9002:9002 owasp/sonarqube || true'*/
          sh 'mvn sonar:sonar -Dsonar.projectKey=devsecops -Dsonar.host.url=http://localhost:9000 -Dsonar.login=3ec9d410b5af187f739beab08f7ccb171cfb48f9'
        }
      }
    
    stage ('Build App') {
      steps {
      sh 'mvn clean package'
       }
    }

    /*...........................DAST................................*/
    stage ('OWASP ZAP - DAST') {
      steps {
        /* sh 'docker run --name webgoat -p 8080:8080 -p 9090:9090 -d webgoat/goatandwolf */
        sh '''
          IPADD=$(ip -f inet -o addr show ens33 | awk '{print $4}' | cut -d '/' -f 1)
          docker run --user $(id -u):$(id -g) -v $(pwd):/zap/wrk/:rw --rm -t owasp/zap2docker-stable zap-baseline.py -t http://${IPADD}:8080/WebGoat/ -r reports/zap-baseline-scan.html || true
          '''
          }
        }
    
       /*.......................Vulnerability Scan....................*/
    stage ('Vulnerability Scan') {
      steps {
        parallel(
          "Vulnerability Scanner for Container Images":{
              /* curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh -s -- -b /usr/local/bin */
              sh 'grype docker.io/webgoat/goatandwolf --file reports/vulnerability-scan-report.json'
          },
          "Trivy Scan":{
              sh "trivy image docker.io/webgoat/webgoat --security-checks vuln > reports/trivy_report"
          }
        )
       }
    }
  }
    post {
        always {
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'reports', reportFiles: 'zap-baseline-scan.html', reportName: 'OWASP ZAP Report', reportTitles: 'OWASP ZAP Report', useWrapperFileDirectly: true])
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'reports', reportFiles: 'vulnerability-scan-report.json', reportName: 'Vulnerability Scan Report', reportTitles: 'Vulnerability Scan Report', useWrapperFileDirectly: true])
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'reports', reportFiles: 'trufflehog.json', reportName: 'Git Secrets Report', reportTitles: 'Git Secrets Report', useWrapperFileDirectly: true])
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'reports', reportFiles: 'trivy_report', reportName: 'Trivy Vulnerability Report', reportTitles: 'Trivy Vulnerability Report', useWrapperFileDirectly: true])
            dependencyCheckPublisher pattern: 'reports/dependency-check-report.xml'
        }
     }
  }