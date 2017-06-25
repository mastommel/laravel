#!groovy

pipeline {
  agent any

  stages {
    stage('Prepare') {
      steps {
        sh 'mkdir -p build/{coverage,html,logs,pdepend,phpdox}'
        sh 'composer install'
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
