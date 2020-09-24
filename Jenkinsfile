pipeline {
  agent any
  stages {
    environment {
      COMPOSE_PROJECT_NAME = 'agripath_2'
    }
    stage('Start Docker') {
      steps {
        sh 'echo "user is: ${USER}"'
        sh 'docker-compose -f docker-compose.yml up -d mysql php-fpm redis workspace nginx'
      }
    }

    stage('Create the database') {
      steps {
        sh 'docker-compose exec -T mysql mysql -u root -proot -e "CREATE DATABASE IF NOT EXISTS agripath;"'
      }
    }

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
        }

        sh 'docker-compose exec -T -w /var/www/api php-fpm chown -R www-data:www-data .'

        dir(path: '../code/app') {
          git 'git@github.com:ldiebold/agripath-app.git'
        }

        dir(path: '../code/admin') {
          git 'git@github.com:ldiebold/agripath-admin.git'
        }

      }
    }

    stage('Install Dependencies') {
      steps {
        sh 'docker-compose exec -T -w /var/www/api workspace composer install --dev'
        sh 'docker-compose exec -T -w /var/www/api workspace yarn'
        sh 'docker-compose exec -T -w /var/www/admin workspace yarn'
        sh 'docker-compose exec -T -w /var/www/app workspace yarn'
        sh 'docker-compose exec -T -w /var/www/api workspace yarn cy:install'
      }
    }

    stage('Build Production Code') {
      steps {
        sh 'docker-compose exec -T -w /var/www/api workspace php artisan key:generate'
        sh 'docker-compose exec -T -w /var/www/app workspace yarn build:pwa'
        sh 'docker-compose exec -T -w /var/www/admin workspace yarn build:pwa'
      }
    }

    stage('Run Test') {
      
      steps {
        sh 'docker-compose exec -T -w /var/www/api workspace yarn test:e2e:CI'
      }
    }

  }
}