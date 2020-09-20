pipeline {
  agent any
  stages {
    stage('Start Docker') {
      steps {
        sh 'docker-compose -f docker-compose.yml -f docker-compose-ci.yml up -d'
      }
    }

    stage('Update Repositories') {
      steps {
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
    }

    stage('Install Dependencies') {
      steps {
          // API
          sh "docker-compose exec -T -w /var/www/api workspace composer install"
          // Admin
          sh "docker-compose exec -T -w /var/www/admin workspace yarn"
          // App
          sh "docker-compose exec -T -w /var/www/app workspace yarn"
      }
    }

    stage('Build Production Code') {
      steps {
        // App
        sh "docker-compose exec -T -w /var/www/app workspace yarn build"
        // Admin
        sh "docker-compose exec -T -w /var/www/admin workspace yarn build"
      }
    }

    stage('Server Production Code') {
      steps {
          // App
          sh "docker-compose -f docker-compose.yml -f docker-compose-ci.yml up -d app"
          // Admin
          sh "docker-compose -f docker-compose.yml -f docker-compose-ci.yml up -d admin"
      }
    }
  }
}