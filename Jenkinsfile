pipeline {
  agent {
    dockerfile {
      filename 'docker-compose.yml'
    }

  }
  stages {
    stage('build') {
      steps {
        dir(path: './laradock') {
          sh 'ls'
        }

      }
    }

  }
}