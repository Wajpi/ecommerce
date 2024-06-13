//def microservices = ['ecomm-web','ecomm-cart', 'ecomm-product', 'ecomm-order']
def microservices = [ 'ecomm-product']
def frontendservice = ['ecomm-front']
//def services = microservices + frontendservice
def services = frontendservice

pipeline {
    agent any
    tools{
        maven 'maven'
    
    }
    
    environment {
        DOCKERHUB_USERNAME = "wajvi"
	SSH_CREDENTIALS_ID = 'ssh-id'
        MASTER_NODE = 'master@master-node'
	BRANCH_NAME = 'test'
	deployenv = 'test'
        
    }
    
    stages{
        
    
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
	   /*

        stage('Build Maven'){
            steps{
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


*/
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
                    for (def service in services) {
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
				for (def service in services) {
					sh "docker push ${DOCKERHUB_USERNAME}/${service}:latest"
                            		//sh "docker rmi -f wajvi/${service}:latest"
                        }
                    }
                }
            }
	   }
	     stage('Test SSH Connection') {
            steps {
                sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                    // Execute a simple command on the remote machine to test SSH connection
                    sh 'ssh -o StrictHostKeyChecking=no $MASTER_NODE "echo SSH connection successful"'
                }
            }
        }
/*
	    stage('Kube-bench Scan') {
          
            steps {
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {
                    sh "ssh $MASTER_NODE 'kube-bench > kubebench_CIS_${BRANCH_NAME}.txt'"
                    sh "ssh $MASTER_NODE cat kubebench_CIS_${BRANCH_NAME}.txt"
                }
            }
        }

	    */
        stage('Kubescape Scan') {
            when {
                expression { (env.BRANCH_NAME == 'test') || (env.BRANCH_NAME == 'master') }
            }
            steps {
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {
                    sh "ssh $MASTER_NODE 'kubescape scan framework mitre -v > kubescape_mitre_${env.BRANCH_NAME}.txt'"
                    sh "ssh $MASTER_NODE cat kubescape_mitre_${env.BRANCH_NAME}.txt"
                }
            }
        }
            
            
           stage('Get YAML Files') {
           
            steps {
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {
                    script {
                        sh "rm -f deploy_to_${deployenv}.sh"
                        sh "wget \"https://raw.githubusercontent.com/Wajpi/ecommerce/test/deploy_to_${deployenv}.sh\""
                        sh "scp deploy_to_${deployenv}.sh $MASTER_NODE:~"
                        sh "ssh $MASTER_NODE chmod +x deploy_to_${deployenv}.sh"
                        sh "ssh $MASTER_NODE ./deploy_to_${deployenv}.sh"
                    }
                }
            }
        }

        stage('Scan YAML Files') {
          
            steps {
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {
                    script {
                        sh "ssh $MASTER_NODE rm -f kubescape_infrastructure_${deployenv}.txt"
                        sh "ssh $MASTER_NODE rm -f kubescape_microservices_${deployenv}.txt"
                        sh "ssh $MASTER_NODE 'kubescape scan ${deployenv}_manifests/infrastructure/*.yml > kubescape_infrastructure_${deployenv}.txt'"
                        sh "ssh $MASTER_NODE 'kubescape scan ${deployenv}_manifests/microservices/*.yml > kubescape_microservices_${deployenv}.txt'"
                    }
                }
            }
        }

    
        stage('Deploy to Kubernetes') {
            steps {
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {
                    script {
                        sh "ssh $MASTER_NODE kubectl apply -f ${deployenv}_manifests/namespace.yml"
                        sh "ssh $MASTER_NODE kubectl apply -f ${deployenv}_manifests/infrastructure/"
                        for (service in microservices) {
                            sh "ssh $MASTER_NODE kubectl apply -f ${deployenv}_manifests/microservices/${service}.yml"
                        }
                    }
                }
            }
        }
    
      
           
       
       
        
        
    }
}
