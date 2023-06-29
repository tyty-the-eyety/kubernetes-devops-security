pipeline {
  agent any
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
	stage('Mutation Tests - PIT') {
      steps {
        sh "mvn org.pitest:pitest-maven:mutationCoverage"
      }
      post {
        always {
          pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
        }
      }
    }
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


  stage('Kubernetes Deployment - DEV') {
      steps {
        sh "sed -i 's#REPLACE_ME#tyron-esch:5000/java-app:latest#g' k8s_deployment_service.yaml"
        sh "kubectl apply -f k8s_deployment_service.yaml"
      }
   }
}}