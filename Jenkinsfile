environment {
    registry = "nicoha/vuln-image"
    registryCredential = 'dockerhub'
}
stages {
  stage('Building image') {
    steps{
      script {
        docker.build registry + ":$BUILD_NUMBER"
      }
    }
  }
}
