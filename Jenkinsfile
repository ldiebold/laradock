pipeline {
  agent any
  stages {
    stage('Prepare') {
      steps {
        step('Start Docker') {
          sh 'docker-compose -f docker-compose.yml -f docker-compose-ci.yml up -d'
        }

        step('Update Repositories') {
          // If the application directories don't exist, create them
          sh "mkdir -p ./code/api"
          sh "mkdir -p ./code/app"
          sh "mkdir -p ./code/admin"

          dir ('./code/api') {
              // Pull the api code
              git url: 'git@github.com:ldiebold/api.git'
          }

          dir ('./code/app') {
              // Pull the app code
              git url: 'git@github.com:ldiebold/agripath-app.git'
          }

          dir ('./code/admin') {
              // Pull the admin code
              git url: 'git@github.com:ldiebold/agripath-admin.git'
          }
        }

        step('Install Dependencies') {
          // API
          sh "docker-compose exec -T -w /var/www/api workspace composer install"
          // Admin
          sh "docker-compose exec -T -w /var/www/admin workspace yarn"
          // App
          sh "docker-compose exec -T -w /var/www/app workspace yarn"
        }

        step('Build Production Code') {
          // App
          sh "docker-compose exec -T -w /var/www/app workspace yarn build"
          // Admin
          sh "docker-compose exec -T -w /var/www/admin workspace yarn build"
        }

        step('Server Production Code') {
            // App
            sh "docker-compose -f docker-compose.yml -f docker-compose-ci.yml up -d app"
            // Admin
            sh "docker-compose -f docker-compose.yml -f docker-compose-ci.yml up -d admin"
        }

        // Generate key
        sh "docker-compose exec -T -w '/var/www/api' workspace php artisan key:generate"
      }
    }

  }
}