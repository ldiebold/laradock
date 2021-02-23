pipeline {
  agent any

  environment {
    COMPOSE_PROJECT_NAME = 'agripath_api'
    docker_compose = 'docker-compose -f docker-compose.yml -f docker-compose-ci.yml'
    api_exec = 'docker-compose -f docker-compose.yml -f docker-compose-ci.yml exec -T -w /var/www/api '
  }

  stages {
    stage('Start Docker Workspace') {
      steps {
        sh '${docker_compose} up -d'
      }
    }

    stage('Setup') {
      steps {
        sh 'mkdir -p ../code/api'
        dir(path: '../code/api') {
          git 'git@github.com:ldiebold/api.git'
          script {
            // if there's no .env file, lets go ahead and create it from the example .env
            if (!fileExists('.env')) {
              sh 'cp .env.cypress .env'
            }
          }
        }

        sh '${api_exec} workspace composer install'
        sh '${api_exec} workspace php artisan key:generate'
        
        sh '${api_exec} workspace yarn'
        sh '${api_exec} workspace yarn cy:install'
      }
    }

    stage('Prepare Docker Containers') {
      steps {
        sh '${api_exec} php-fpm chown -R www-data:www-data .'
      }
    }

    stage('Run Tests') {
      environment { COMPOSE_PROJECT_NAME = 'agripath_1' }
      steps {
        sh 'docker-compose exec -T -w /var/www/api workspace yarn test:e2e:CI'
      }
    }
  }
}