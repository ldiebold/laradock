#!/usr/bin/env groovy

node('master') {
    environment {
        DISABLE_AUTH = 'true'
        DB_ENGINE    = 'sqlite'
    }

    stage('build') {

        dir ('./laradock') {
            // Pull in laradock
            git url: 'git@github.com:ldiebold/laradock.git'

            // Start services (Let docker-compose build containers for testing)
            sh "docker-compose -f docker-compose.yml -f docker-compose-ci.yml up -d"
        }

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

        // API Setup
        dir ('./laradock') {
            // Install dependencies
            sh "docker-compose exec -T -w /var/www/api workspace composer install"
            // Generate key
            sh "docker-compose exec -T -w '/var/www/api' workspace php artisan key:generate"
        }

        // App Setup
        dir ('./laradock') {
            // Install dependencies
            sh "docker-compose exec -T -w /var/www/app workspace yarn"
            // Build production code
            sh "docker-compose exec -T -w /var/www/app workspace yarn build"
            // Serve production code!
            sh "docker-compose run -w /var/www/app app yarn serve -p 9091"
        }

        // Admin Setup
        dir ('./laradock') {
            // Install dependencies
            sh "docker-compose exec -T -w /var/www/admin workspace yarn"
            // Build production code
            sh "docker-compose exec -T -w /var/www/admin workspace yarn build"
            // Serve production code!
            sh "docker-compose run -w /var/www/admin admin yarn serve -p 9092"
        }
    }
    // stage('test') {
    //     sh "APP_ENV=testing ./develop test"
    // }
}

// Start web server

// Wait for web server to response

// Run cypress tests

// Stop web server