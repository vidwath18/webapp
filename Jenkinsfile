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
	    
  	stage ('Run Trivy') {
		    steps {
	        sh 'trivy repo https://github.com/vidwath18/webapp.git > trivy'
		sh 'cat trivy'
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
	    
	    stage ('SSL Checks') {
		    steps {
			sh 'pip install sslyze==1.4.2'
			sh 'python -m sslyze --regular 18.157.159.70 --json_out sslyze-output.json'
			sh 'cat sslyze-output.json'
		    }
	    }
	    
	    stage ('Upload Reports to Defect Dojo') {
		    steps {
			sh 'pip install requests'
			sh 'wget https://raw.githubusercontent.com/vidwath18/webapp/master/upload-results.py'
			sh 'chmod +x upload-results.py'
			sh 'python upload-results.py --host 18.157.159.70:80 --api_key 66879c160803596f132aff025fee9a170366f615 --engagement_id 4 --result_file trufflehog --username admin --scanner "SSL Labs Scan"'
			sh 'python upload-results.py --host 18.157.159.70:80 --api_key 66879c160803596f132aff025fee9a170366f615 --engagement_id 4 --result_file nmap --username admin --scanner "Nmap Scan"'
			sh 'python upload-results.py --host 18.157.159.70:80 --api_key 66879c160803596f132aff025fee9a170366f615 --engagement_id 4 --result_file sslyze-output.json --username admin --scanner "SSL Labs Scan"'
			sh 'python upload-results.py --host 18.157.159.70:80 --api_key 66879c160803596f132aff025fee9a170366f615 --engagement_id 4 --result_file nikto-output.xml --username admin'
			    
		    }
	    }
	    
	
    }
}
