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

          // Install api packages
          sh "docker-compose exec -T -w /var/www/api workspace yarn"

        dir ('./code/app') {
            // Pull the app code
            git url: 'git@github.com:ldiebold/agripath-app.git'
        }

        dir ('./code/admin') {
            // Pull the admin code
            git url: 'git@github.com:ldiebold/agripath-admin.git'
        }

        // API Setup
        // Install dependencies
        sh "docker-compose exec -T -w /var/www/api workspace composer install"
        // Generate key
        sh "docker-compose exec -T -w '/var/www/api' workspace php artisan key:generate"
      }
    }

  }
}