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
              -Dsonar.host.url=http://192.168.0.27:9000 -Dsonar.token=sqp_d69737c7f775feff548a521d2d0243c7b373a7ab""
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