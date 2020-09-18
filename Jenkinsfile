#!/usr/bin/env groovy

node('master') {
    stage('build') {

        dir ('./laradock') {
            // Pull in laradock
            git url: 'git@github.com:ldiebold/laradock.git'

            // Create .env file for testing
            sh 'cp .ci.env .env'

            // Start services (Let docker-compose build containers for testing)
            sh "docker-compose -f docker-compose.yml -f docker-compose-ci.yml up -d"
        }


        dir ('./code') {
            // Pull the api git repo
            git url: 'git@github.com:ldiebold/api.git api'
        }

        dir ('./laradock') {
            // composer install
            sh "docker-compose exec -w /var/www/api workspace composer install"
            // Generate key
            sh "docker-compose exec -w '/var/www/api' workspace php artisan key:generate"
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