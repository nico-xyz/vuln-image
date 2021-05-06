pipeline {
  environment {
    def app
    registry = "nicoha/vuln-image"
    registryCredential = "dockerhub"
    dockerImage = ""
  }
  agent any
    stage("1") {
      app = docker.build("centos")
      echo "Loading image.."
    }
    stage("2") {
      steps{
            sh '''
            #!/bin/bash
            rpm -ivh https://github.com/aquasecurity/trivy/releases/download/v0.16.0/trivy_0.16.0_Linux-64bit.rpm"
            echo "Installing Trivy scanner..."
            trivy --exit-code 1 --severity CRITICAL nicoha/vuln-image
            echo "Scanning image for critical vulnerabilities..."
         '''
        script {
          docker.build registry + ":$BUILD_NUMBER"
        }
      }
      stage('Push image') {
        docker.withRegistry('https://registry.hub.docker.com', 'git') {            
       app.push("${env.BUILD_NUMBER}")            
       app.push("latest")        
              }    
           }
    }
}
