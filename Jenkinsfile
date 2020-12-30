pipeline {
  agent any
  stages {
    stage('Start Docker Workspace') {
      steps {
        sh 'echo "Success!!!"'
      }
    }

  }
  environment {
    COMPOSE_PROJECT_NAME = 'agripath_api'
    docker_compose = 'docker-compose -f docker-compose.yml -f docker-compose-ci.yml'
    api_exec = 'docker-compose -f docker-compose.yml -f docker-compose-ci.yml exec -T -w /var/www/api '
  }
}