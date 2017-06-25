#!groovy

pipeline {
  agent any

  environment {
    APP_ENV = 'testing'
    CACHE_DRIVER = 'array'
    SESSION_DRIVER = 'array'
    QUEUE_DRIVER = 'sync'
    APP_DEBUG = 'true'
    APP_LOG_LEVEL = 'debug'
    DB_CONNECTION = 'sqlite'
    DB_DATABASE = 'build/database.sqlite'
  }
  
  stages {
    stage('Prepare') {
      steps {
        sh 'rm -rf build/'
        sh 'mkdir -p build/coverage'
        sh 'mkdir -p build/html'
        sh 'mkdir -p build/logs'
        sh 'mkdir -p build/pdepend'
        sh 'mkdir -p build/phpdox'
        sh 'composer install --no-ansi --no-interaction --prefer-dist --optimize-autoloader'
        sh 'cp .env.example .env'
        sh '> {$env.DB_DATABASE}'
        sh 'php artisan key:gen --no-ansi'
        sh 'php artisan migrate --env=testing --no-ansi'
        sh 'php artisan db:seed --env=testing --no-ansi'
      }
    }
    stage('php-lint') {
      steps {
        sh 'vendor/bin/parallel-lint --exclude vendor .'
      }
    }
    stage('php-loc') {
      steps {
        sh 'vendor/bin/phploc --count-tests --log-csv build/logs/phploc.csv --log-xml build/logs/phploc.xml --exclude=vendor .'
      }
    }
    stage('php-unit') {
      steps {
        sh 'vendor/bin/phpunit --configuration phpunit.xml --coverage-html build/coverage --coverage-clover build/logs/clover.xml --coverage-crap4j build/logs/crap4j.xml --log-junit build/logs/junit.xml'
      }
    }
    stage('php-depend') {
      steps {
        sh 'vendor/bin/pdepend --jdepend-xml=build/logs/jdepend.xml --jdepend-chart=build/pdepend/dependencies.svg --overview-pyramid=build/pdepend/overview-pyramid.svg --ignore=vendor .'
      }
    }
    stage('php-md') {
      steps {
        sh 'vendor/bin/phpmd . text phpmd.xml'
      }
    }
    stage('php-dox') {
      steps {
        sh 'vendor/bin/phpdox'
      }
    }
    stage('php-cs') {
      steps {
        sh 'vendor/bin/phpcs'
      }
    }
    stage('php-cpd') {
      steps {
        sh 'vendor/bin/phpcpd .'
      }
    }
  }
}
