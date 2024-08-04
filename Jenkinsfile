pipeline {
  agent any
  tools { 
        maven 'Maven_3_5_2'  
    }
stages{
       stage('CompileandRunSonarAnalysis') {
            steps {	
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=asgbuggywebappraezon -Dsonar.organization=asgbuggywebappraezon -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=1cc2143703da2f6a000facf8727c484101a82c77'
			}
    }

	stage('RunSCAAnalysisUsingSnyk') {
            steps {		
		   withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
					sh 'mvn snyk:test -fn'
				}
			}
    }	

	stage('Build') { 
            steps { 
               withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                 script{
                 app =  docker.build("asg")
                 }
               }
            }
    }

	stage('Push') {
            steps {
                script{
                    docker.withRegistry('https://280415466858.dkr.ecr.eu-central-1.amazonaws.com', 'ecr:eu-central-1:aws-credentials') {
                    app.push("latest")
                    }
                }
            }
    	}
	   
	stage('Kubernetes Deployment of ASG Bugg Web Application') {
	   steps {
	      withKubeConfig([credentialsId: 'kubelogin']) {
		  sh('kubectl delete all --all -n devsecops')
		  sh ('kubectl apply -f deployment.yaml --namespace=devsecops')
		}
	      }
   	}

  }
}
