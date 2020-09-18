#!/usr/bin/env groovy

node('master') {
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
            // Pull the api git repo
            git url: 'git@github.com:ldiebold/api.git'
        }

        dir ('./code/app') {
            // Pull the app git repo
            git url: 'git@github.com:ldiebold/agripath-app.git'
        }

        dir ('./code/admin') {
            // Pull the admin git repo
            git url: 'git@github.com:ldiebold/agripath-admin.git'
        }

        dir ('./laradock') {
            // composer install
            sh "docker-compose exec -T -w /var/www/api workspace composer install"
            // Generate key
            sh "docker-compose exec -T -w '/var/www/api' workspace php artisan key:generate"
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