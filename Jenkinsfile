pipeline {
  agent any
  stages {
    stage('build') {
      steps {
        sh 'docker-compose -f docker-compose.yml -f docker-compose-ci.yml up -d'
      }
    }

  }
}