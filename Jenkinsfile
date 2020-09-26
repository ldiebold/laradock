pipeline {
  agent {
    node {
      label 'tester'
    }

  }
  stages {
    stage('Start Docker Workspace') {
      steps {
        sh '${docker_compose} up -d workspace'
      }
    }

    stage('Setup API Code') {
      steps {
        sh 'mkdir -p ../code/api'
        dir(path: '../code/api') {
          git 'git@github.com:ldiebold/api.git'
          script {
            if (!fileExists('.env')) {
              sh 'cp .env.cypress .env'
            }
          }

        }

        sh '${docker_compose} exec -T -w /var/www/api workspace composer install'
        sh '${docker_compose} exec -T -w /var/www/api workspace yarn'
        sh '${docker_compose} exec -T -w /var/www/api workspace yarn cy:install'
        sh '${docker_compose} exec -T -w /var/www/api workspace php artisan key:generate'
      }
    }

    stage('Setup Admin Code') {
      steps {
        sh 'mkdir -p ../code/admin'
        dir(path: '../code/admin') {
          git 'git@github.com:ldiebold/agripath-admin.git'
        }

        sh '${docker_compose} exec -T -w /var/www/admin workspace yarn'
      }
    }

    stage('Setup App Code') {
      steps {
        sh 'mkdir -p ../code/app'
        dir(path: '../code/app') {
          git 'git@github.com:ldiebold/agripath-app.git'
        }

        sh '${docker_compose} exec -T -w /var/www/app workspace yarn'
      }
    }

    stage('Prepare Docker Containers') {
      steps {
        sh '${docker_compose} down'
        sh 'COMPOSE_PROJECT_NAME=agripath_1 ${docker_compose} up -d mysql php-fpm redis workspace nginx'
        sh 'COMPOSE_PROJECT_NAME=agripath_1 ${docker_compose} exec -T mysql mysql -u root -proot -e "CREATE DATABASE IF NOT EXISTS agripath;"'
        sh 'COMPOSE_PROJECT_NAME=agripath_1 ${docker_compose} exec -T -w /var/www/api php-fpm chown -R www-data:www-data .'
        sh 'COMPOSE_PROJECT_NAME=agripath_2 ${docker_compose} up -d mysql php-fpm redis workspace nginx'
        sh 'COMPOSE_PROJECT_NAME=agripath_2 ${docker_compose} exec -T mysql mysql -u root -proot -e "CREATE DATABASE IF NOT EXISTS agripath;"'
      }
    }

    stage('Run Tests') {
      parallel {
        stage('Run tests agripath_1') {
          environment {
            COMPOSE_PROJECT_NAME = 'agripath_1'
          }
          steps {
            sh 'docker-compose exec -T -w /var/www/api workspace yarn test:e2e:CI'
          }
        }

        stage('Run tests agripath_2') {
          environment {
            COMPOSE_PROJECT_NAME = 'agripath_2'
          }
          steps {
            sh 'docker-compose exec -T -w /var/www/api workspace yarn test:e2e:CI'
          }
        }

      }
    }

  }
  environment {
    docker_compose = 'docker-compose -f docker-compose.yml -f docker-compose-ci.yml'
  }
}