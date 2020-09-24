pipeline {
  agent any
  stages {
    stage('Start Docker') {
      steps {
        sh 'docker-compose -f docker-compose.yml up -d mysql php-fpm redis workspace nginx'
      }
    }

    stage('Update Repositories') {
      steps {
        sh 'mkdir -p ../code/api'
        sh 'mkdir -p ../code/app'
        sh 'mkdir -p ../code/admin'
        dir(path: '../code/api') {
          git 'git@github.com:ldiebold/api.git'
          sh 'chgrp -R www-data storage'
        }

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