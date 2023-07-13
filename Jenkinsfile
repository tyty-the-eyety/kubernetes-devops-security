pipeline {
  agent any
  
  environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "docker-registry:5000/java-app:latest"
    applicationURL = "http://controlplane:30010"
    applicationURI = "/increment/99"
  }
  
  stages {
    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }
	
    stage('Unit Tests - JUnit and Jacoco') {
      steps {
        sh "mvn test"
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
		  sh 'docker logout'
        }
      }
	  
	  
    }
	
	/*stage('sonar qube tests - SAST') {
      steps {
        sh "mvn clean verify sonar:sonar   -Dsonar.projectKey=numeric-application   -Dsonar.projectName='numeric-application'   -Dsonar.host.url=http://192.168.0.27:9000   -Dsonar.token=sqp_d69737c7f775feff548a521d2d0243c7b373a7ab"
      }

	  
	  
    }*/
	
	stage('SonarQube - SAST') {
      steps {
        withSonarQubeEnv('sonarqube') {
          sh "mvn sonar:sonar \
              -Dsonar.projectKey=devsecops-numeric-application \
              -Dsonar.host.url=http://192.168.0.27:9000 "
      }
        timeout(time: 2, unit: 'MINUTES') {
          script {
            waitForQualityGate abortPipeline: true
          }
        }
      }
    }
	
	//stage('Mutation Tests - PIT') {
    //  steps {
    //    sh "mvn org.pitest:pitest-maven:mutationCoverage"
    //  }
    //  post {
    //    always {
    //      pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
    //    }
    //  }
    //}
	/*
    stage('Docker image build and push') {
      steps {withDockerRegistry([ credentialsId: "dockerHub", url: "" ])
        //sh 'docker build -t docker-registry:5000/java-app:latest .'
        //sh 'docker push docker-registry:5000/java-app:latest'
		//sh 'docker ps -a'
		dockerImage = docker.build("tyronesch/java-app:latest")
		dockerImage.push()
      }
    }
	*/
	
/*    stage('Vulnerability Scan - Docker ') {
      steps {
        sh "mvn dependency-check:check"
      }
      post {
        always {
          dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
        }
      }
    }
*/
    stage('Vulnerability Scan - Docker') {
      steps {
        parallel(
          "Dependency Scan": {
            sh "mvn dependency-check:check"
          },
          "Trivy Scan": {
            //sh "bash trivy-docker-image-scan.sh"
			sh "echo 'skip trivy scan cause of rate limit failures' || exit 0"
          }
        )
      }
    }
	
	 stage('Docker image build and push') {
		 steps {
			withDockerRegistry([ credentialsId: "dockerHub", url: "" ]) {
			//dockerImage = docker.build("tyronesch/java-app:latest")
			
			sh 'docker build -t tyronesch/java-app:latest .'
			sh 'docker push tyronesch/java-app:latest'
			//dockerImage.push()
			}
		 }

  }
	  
	 stage('Vulnerability Scan - Kubernetes') {
      steps {
        sh '/usr/local/bin/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
      }
     }
	  

      stage('K8S Deployment - DEV') {
      steps {
        parallel(
          "Deployment": {
              sh "bash k8s-deployment.sh"
          },
          "Rollout Status": {
              sh "bash k8s-deployment-rollout-status.sh"
          }
        )
      }
    }
   stage('Integration Tests - DEV') {
      steps {
        script {
          try {
            sh "bash integration-test.sh"
            }
          catch (e) {
            sh "kubectl -n default rollout undo deploy ${deploymentName}"
          throw e
          }
        }
      }
    }
	
	stage('OWASP ZAP - DAST') {
      steps {
        sh 'bash zap.sh'
      }
    }
	post {
    always {
     publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'owasp-zap-report', reportFiles: 'zap_report.html', reportName: 'OWASP ZAP HTML Reports', reportTitles: 'OWASP ZAP HTML Reports'])
    }
  }
}