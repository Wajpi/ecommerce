def microservices = ['ecomm-cart', 'ecomm-db', 'ecomm-ui','ecomm-user']
//def microservices = ['ecomm-cart', 'ecomm-product', 'ecomm-order' ]


pipeline {
    agent any
    tools{
        maven 'maven'
    
    }
    
    environment {
        DOCKERHUB_USERNAME = "wajvi"
        
    }
    
    stages{
        /*
         stage('Deploy to Kubernetes') {
            
            when {
                expression { (env.BRANCH_NAME == 'test') || (env.BRANCH_NAME == 'master') }
            }
            steps {
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {
                    script {
                        if (env.BRANCH_NAME == 'test') {
                            sh "ssh $MASTER_NODE kubectl apply -f cart.yml"
 			   sh "ssh $MASTER_NODE kubectl apply -f namespace.yml"                        
		} else if (env.BRANCH_NAME == 'master') {
                            sh "ssh $MASTER_NODE kubectl apply -f cart.yml"
 			   sh "ssh $MASTER_NODE kubectl apply -f namespace.yml"                        
 			}
                    }
                }
            }
        }
        */
    
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }
        stage("Checkout from SCM"){
                steps {
                    //git branch: 'main', credentialsId: 'github', url: 'https://github.com/Wajpi/ecommerce.git'
                    git branch: 'main', changelog: false, poll: false, url: 'https://github.com/Wajpi/ecommerce.git'
                }
        }

	    /*
         stage("OWASP Dependency Check"){
                steps {
            
                dependencyCheck additionalArguments: '--format HTML', odcInstallation: 'DP-check'
                //sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
                
                }
         }
	 */
	    

        stage('Build Maven'){
            steps{
                //checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Wajpi/ecommerce.git']])
              // sh 'cd ecomm-product && mvn clean install'
		script {
                    for (def service in microservices) {
                        dir(service) {
                            sh 'mvn clean install'
                        }
                    }
                }
            }
        }
        
        stage('Unit test'){
            steps{
                //checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Wajpi/ecommerce.git']])
                //sh "cd ecomm-product && mvn test"
		    script {
                    for (def service in microservices) {
                        dir(service) {
                            sh 'mvn test'
                        }
                    }
                }
            }
        }
        
  /*
        stage("SonarQube Analysis"){
           steps {
	           script {
		        withSonarQubeEnv(credentialsId: 'sonarqube-token') { 
                        //sh "cd ecomm-product && mvn sonar:sonar"
	                    for (def service in microservices) {
	                        dir(service) {
	                            sh 'mvn sonar:sonar'
                        		}
                   		 }
		        }
	           }	
           }
       }
       /*
	    
       stage("Quality Gate"){
                   steps {
                       script {
                            waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-token'
                        }	
                    }
        
                }
		/*
	     stage("Trivy Image Scan"){
                steps{
                    script{
                        sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v $PWD:/tmp/.cache/ aquasec/trivy image --scanners vuln --timeout 30m wajvi/ecomm-product-1.0:latest > trivy-ecomm.txt"

                    }
                }
            }
	    */
	    
       stage("Docker Build"){
           steps{
               script{
                   //sh 'cd ecomm-product && docker build -t wajvi/ecomm-product   .'
                    for (def service in microservices) {
                        dir(service) {
				sh "docker build -t wajvi/${service}:latest ."
                       			}
                   		}
                        }
                    }
                 }
	    
           stage("Docker login and Push"){
               steps{
                   script{
                       withCredentials([string(credentialsId: 'dockerhub-pwd1', variable: 'dockerhubpwd')]) {
                           sh "docker login -u wajvi -p ${dockerhubpwd} "
                           //sh "docker push wajvi/ecomm-product"
				for (def service in microservices) {
					sh "docker push ${DOCKERHUB_USERNAME}/${service}:latest"
                            		//sh "docker rmi -f wajvi/${service}:latest"
                        }
                    }
                }
            }
	   }
            
            
                
           /*
          
            stage('Deploy to Kubernetes') {
            steps {
                script {
                    kubernetesDeploy(kubeconfigId: 'id', configs: 'k8s/kubernetes.yaml')
                
                }
            }
        }
        */
    
      
           
       
       
        
        
    }
}
