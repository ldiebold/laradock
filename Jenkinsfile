pipeline {
  agent "any"
  stages {
    stage('Build') {
      parallel {
        stage('build app') {
          agent {
              label "agent1"
          }
          steps {
              sh "echo 'app built by $NODE_NAME'"
          }
        }
      }
    }
  }
  environment {
    COMPOSE_PROJECT_NAME = 'agripath_api'
    docker_compose = 'docker-compose -f docker-compose.yml -f docker-compose-ci.yml'
    api_exec = 'docker-compose -f docker-compose.yml -f docker-compose-ci.yml exec -T -w /var/www/api '
  }
}