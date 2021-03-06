pipeline {
    agent any

    tools {
        maven 'Maven'
        nodejs 'nodejs'
    }
  
    stages {
        stage('initial'){
            steps{
                figlet 'Inital'
                
             sh '''
              echo "PATH = ${PATH}"
              echo "M2_HOME = ${M2_HOME}"
              '''
            }
        }

        stage('Compile'){
            steps{
                figlet 'Compile'
                sh 'mvn clean compile -e'
            }
        }
        
        stage('Test'){
            steps{
                figlet 'Test'
                sh 'mvn clean test -e'
            }
        }
        
      
        
       
        
        stage('SCA'){
            steps{
                figlet 'Dependency-Check'
                sh 'mvn org.owasp:dependency-check-maven:check'
                
                archiveArtifacts artifacts: 'target/dependency-check-report.html', followSymlinks: false
                     }
        }
                
        stage('Scan Docker'){
               steps{
                 figlet 'Scan Docker'
               script{
                  env.DOCKER = tool "Docker"
        				         env.DOCKER_EXEC = "${DOCKER}/bin/docker"

                sh '''
                   ${DOCKER_EXEC} run --rm -v $(pwd):/root/.cache/ aquasec/trivy openjdk:8-jdk-alpine
                '''

                sh '${DOCKER_EXEC} rmi aquasec/trivy'
                
                    		        }
                    			}
            }
       
              stage('DAST'){
            steps{
                figlet 'Owasp Zap DAST'
        		
        		script{
        		   env.DOCKER = tool "Docker"
        		   env.DOCKER_EXEC = "${DOCKER}/bin/docker"
        		   env.TARGET = 'http://zero.webappsecurity.com'        	
        		
        		    sh "${DOCKER_EXEC} pull owasp/zap2docker-stable"
                    sh '${DOCKER_EXEC} run --add-host="localhost:192.168.1.86" --rm -e LC_ALL=C.UTF-8 -e LANG=C.UTF-8 --name zap2 -u zap -p 8094:8094 -d owasp/zap2docker-stable zap.sh -daemon -port 8094 -host 192.168.1.86 -config api.disablekey=true'
                    sh '${DOCKER_EXEC} run --user $(id -u):$(id -g) --add-host="localhost:192.168.1.86" -v /var/jenkins_home:/zap/wrk/:rw --rm -i owasp/zap2docker-stable zap-baseline.py -t "http://zero.webappsecurity.com" -I -r zap_baseline_report.html -l PASS'	
        		   
        		   publishHTML([
        				    allowMissing: false,
        				    alwaysLinkToLastBuild: false,
        				    keepAll: false,
        				    reportDir: '/var/jenkins_home/tools/',
        				    reportFiles: 'zap_baseline_report.html',
        				    reportName: 'HTML Report',
        				    reportTitles: ''])
        		}
            }
        }
          }
          }
