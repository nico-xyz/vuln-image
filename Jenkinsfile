pipeline {
    agent {
        docker { image 'nicoha/vuln-image' }
    }
    stages {
        stage('Test') {
            steps {
                sh 'node --version'
            }
        }
    }
}
