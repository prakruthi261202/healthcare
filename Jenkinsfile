pipeline {
  agent any
  
  tools {
    maven 'M2_HOME' // Ensure this tool is configured in Jenkins
  }
  
  stages {
    stage('Git Checkout') {
      steps {
        echo 'This stage is to clone the repo from GitHub'
        git branch: 'master', url: 'https://github.com/prakruthi261202/healthcare.git' // Check if 'master' is correct
      }
    }
    
    stage('Create Package') {
      steps {
        echo 'This stage will compile, test, and package my application'
        sh 'mvn package' // Ensure this runs successfully and generates reports
      }
    }
    
    stage('Generate Test Report') {
      steps {
        echo 'This stage generates Test report using TestNG'
        publishHTML([
          allowMissing: false, 
          alwaysLinkToLastBuild: false, 
          keepAll: false, 
          reportDir: 'target/surefire-reports', // Use relative path
          reportFiles: 'index.html', // Ensure this file is generated
          reportName: 'HTML Report', 
          reportTitles: '', 
          useWrapperFileDirectly: true
        ])
      }
    }
    
    stage('Create Docker Image') {
      steps {
        echo 'This stage will create a Docker image'
        sh 'docker build -t prakruthi810/healthcare:1.0 .' // Ensure Dockerfile is in the correct location
      }
    }
    
    stage('Login to Dockerhub') {
      steps {
        echo 'This stage will log into Dockerhub'
        withCredentials([usernamePassword(credentialsId: 'Dockerlogin', passwordVariable: 'docker-pass', usernameVariable: 'docker-login')]) {
          sh 'echo "${docker-pass}" | docker login -u "${docker-login}" --password-stdin' // Use safer login method
        }
      }
    }
    
    stage('Docker Push-Image') {
      steps {
        echo 'This stage will push my new image to Dockerhub'
        sh 'docker push prakruthi810/healthcare:1.0'
      }
    }
  }
}
