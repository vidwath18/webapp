pipeline {
    agent any
    tools { 
        maven 'Maven' 
    }
    stages {
        stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                ''' 
            }
        }
	    
	    stage ('Check-Git-Secrets') {
		    steps {
	        sh 'rm trufflehog || true'
		sh 'docker pull gesellix/trufflehog'
		sh 'docker run -t gesellix/trufflehog --json https://github.com/vidwath18/webapp.git > trufflehog'
		sh 'cat trufflehog'
	    }
	    }
	    

	  stage ('Build') {
            steps {
                sh 'mvn clean package'
            }
        }    
	    
  	stage ('Source-Composition-Analysis') {
		steps {
		     sh 'rm synk-* || true'
		     sh 'snykSecurity snykInstallation: 'Please define a Snyk installation in the Jenkins Global Tool Configuration. This task will not run without a Snyk installation.', snykTokenId: 'snyk'
			}
	    }
	}
	    
       stage ('Deploy-To-Tomcat') {
            steps {
           sshagent(['tomcat']) {
                sh 'scp -o StrictHostKeyChecking=no target/*.war ubuntu@18.157.159.70:/prod/apache-tomcat-8.5.56/webapps/webapp.war'
              }      
           }       
    }
	 
	    stage ('Port Scan') {
		    steps {
			sh 'rm nmap* || true'
			sh 'docker run --rm -v "$(pwd)":/data uzyexe/nmap -sS -sV -oX nmap 18.157.159.70'
			sh 'cat nmap'
		    }
	    }
	    
	    stage ('DAST') {
		  
		    	steps {
			    sshagent(['zap']) {
				    sh 'ssh -o StrictHostKeyChecking=no ubuntu@18.185.211.172 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://18.157.159.70:8080/webapp/" || true'
			    }
			}
		}    
	
	    stage ('Nikto Scan') {
		    steps {
			sh 'rm nikto-output.xml || true'
			sh 'docker pull secfigo/nikto:latest'
			sh 'docker run --user $(id -u):$(id -g) --rm -v $(pwd):/report -i secfigo/nikto:latest -h 18.157.159.70 -p 8080 -output /report/nikto-output.xml'
			sh 'cat nikto-output.xml'   
		    }
	    }
	    
	   
	    
	    stage ('Upload Reports to Defect Dojo') {
		    steps {
			sh 'pip install requests'
			sh 'wget https://raw.githubusercontent.com/vidwath18/webapp/master/upload-results.py'
			sh 'chmod +x upload-results.py'
			sh 'python upload-results.py --host 18.157.159.70:80 --api_key 66879c160803596f132aff025fee9a170366f615 --engagement_id 4 --result_file trufflehog --username admin --scanner "SSL Labs Scan"'
			sh 'python upload-results.py --host 18.157.159.70:80 --api_key 66879c160803596f132aff025fee9a170366f615 --engagement_id 4 --result_file /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml --username admin --scanner "Dependency Check Scan"'
			sh 'python upload-results.py --host 18.157.159.70:80 --api_key 66879c160803596f132aff025fee9a170366f615 --engagement_id 4 --result_file nmap --username admin --scanner "Nmap Scan"'
			sh 'python upload-results.py --host 18.157.159.70:80 --api_key 66879c160803596f132aff025fee9a170366f615 --engagement_id 4 --result_file nikto-output.xml --username admin'
			    
		    }
	    }
	
    }
}
