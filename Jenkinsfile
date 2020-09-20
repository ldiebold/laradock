pipeline {
  agent none
  stages {
    stage('Start Docker') {
      steps {
        sh 'docker-compose -f docker-compose.yml -f docker-compose-ci.yml up -d'
        node(label: 'My Node') {
          sh 'docker-compose -f docker-compose.yml -f docker-compose-ci.yml up -d'
        }

      }
    }

  }
}