pipeline {
  agent any
  tools { 
        maven 'Maven_3_5_2'  
    }
stages{
    stage('CompileandRunSonarAnalysis') {

	    steps {
                withCredentials([
                    string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN'),
                    string(credentialsId: 'PROJECT_KEY', variable: 'PROJECT_KEY'),
                    string(credentialsId: 'ORGANIZATION', variable: 'ORGANIZATION'),
                ]) {
                    sh """
                        mvn clean verify sonar:sonar \
                            -Dsonar.projectKey=${PROJECT_KEY} \
                            -Dsonar.organization=${ORGANIZATION} \
                            -Dsonar.token=${SONAR_TOKEN} \
                            -Dsonar.host.url=https://sonarcloud.io
                    """
                }
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
                    docker.withRegistry('ECR_URL', 'ecr:eu-central-1:aws-credentials') {
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
