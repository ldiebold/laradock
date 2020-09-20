pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        sh 'docker-compose -f docker-compose.yml -f docker-compose-ci.yml up -d'

        // If the application directories don't exist, create them
        sh "mkdir -p ./code/api"
        sh "mkdir -p ./code/app"
        sh "mkdir -p ./code/admin"

        dir ('./code/api') {
            // Pull the api code
            git url: 'git@github.com:ldiebold/api.git'
        }
      }
    }

  }
}