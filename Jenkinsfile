pipeline {
  agent any
  stages {
    stage('Update Repositories') {
      steps {
        sh 'mkdir -p ../code/api'
        sh 'mkdir -p ../code/app'
        sh 'mkdir -p ../code/admin'
        dir(path: '../code/api') {
          git 'git@github.com:ldiebold/api.git'
          // www-data needs ownership of storage so logs can be created
          script {
            // if there's no .env file, lets go ahead and create it from the example .env
            if (!fileExists('.env')) {
              sh 'cp .env.cypress .env'
            }
          }

          sh 'docker-compose up -d workspace'

          sh 'docker-compose exec -T -w /var/www/api workspace composer install'
          sh 'docker-compose exec -T -w /var/www/api workspace yarn'
          sh 'docker-compose exec -T -w /var/www/api workspace yarn cy:install'
          sh 'docker-compose exec -T -w /var/www/api workspace php artisan key:generate'

          sh 'docker-compose exec -T -w /var/www/admin workspace yarn'
          sh 'docker-compose exec -T -w /var/www/admin workspace yarn build:pwa'

          sh 'docker-compose exec -T -w /var/www/app workspace yarn'
          sh 'docker-compose exec -T -w /var/www/app workspace yarn build:pwa'
        }

        dir(path: '../code/app') {
          git 'git@github.com:ldiebold/agripath-app.git'
        }

        dir(path: '../code/admin') {
          git 'git@github.com:ldiebold/agripath-admin.git'
        }
      }
    }

    stage('Prepare Docker Containers') {
      steps {
        sh 'docker-compose down'

        sh 'COMPOSE_PROJECT_NAME=agripath_1 docker-compose -f docker-compose.yml up -d mysql php-fpm redis workspace nginx'
        sh 'COMPOSE_PROJECT_NAME=agripath_1 docker-compose exec -T mysql mysql -u root -proot -e "CREATE DATABASE IF NOT EXISTS agripath;"'
        sh 'COMPOSE_PROJECT_NAME=agripath_1 docker-compose exec -T -w /var/www/api php-fpm chown -R www-data:www-data .'
        sh 'COMPOSE_PROJECT_NAME=agripath_2 docker-compose -f docker-compose.yml up -d mysql php-fpm redis workspace nginx'
        sh 'COMPOSE_PROJECT_NAME=agripath_2 docker-compose exec -T mysql mysql -u root -proot -e "CREATE DATABASE IF NOT EXISTS agripath;"'
        sh 'COMPOSE_PROJECT_NAME=agripath_2 docker-compose exec -T -w /var/www/api php-fpm chown -R www-data:www-data .'
      }
    }

    stage('Run Tests') {
      parallel {
        stage('Run tests agripath_1') {
          environment { COMPOSE_PROJECT_NAME = 'agripath_1' }
          steps {
            sh 'docker-compose exec -T -w /var/www/api workspace yarn test:e2e:CI'
            sh 'docker-compose exec -T -w /var/www/api workspace yarn test:e2e:CI'
          }
        }

        stage('Run tests agripath_2') {
          environment { COMPOSE_PROJECT_NAME = 'agripath_2' }
          steps {
            sh 'docker-compose exec -T -w /var/www/api workspace yarn test:e2e:CI'
            sh 'docker-compose exec -T -w /var/www/api workspace yarn test:e2e:CI'
          }
        }
      }
    }
  }
}