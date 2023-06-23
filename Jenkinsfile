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
        withDockerRegistry([ credentialsId: "dockerhubaccount", url: "" ]) {
		dockerImage = docker.build("tyronesch/java-app:latest")
        dockerImage.push()
        }

  }
}}