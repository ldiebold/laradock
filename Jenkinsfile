pipeline {
  agent {
    node {
      label 'Docker Compose'
    }

  }
  stages {
    stage('build') {
      steps {
        dir(path: './laradock') {
          sh 'docker-compose -f docker-compose.yml -f docker-compose-ci.yml up -d'
        }

      }
    }

  }
}