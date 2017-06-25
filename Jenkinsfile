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
    stage('Build Environment') {
      steps {
        sh 'rm -rf build/'
        sh 'mkdir -p build/coverage'
        sh 'mkdir -p build/html'
        sh 'mkdir -p build/logs'
        sh 'mkdir -p build/pdepend'
        sh 'mkdir -p build/phpdox'
        sh 'touch ${DB_DATABASE}'
        sh 'composer install --no-ansi --no-interaction --prefer-dist --optimize-autoloader'
        sh 'cp .env.example .env'
        sh 'php artisan key:gen --no-ansi'
        sh 'php artisan migrate --env=testing --no-ansi'
        sh 'php artisan db:seed --env=testing --no-ansi'
      }
    }

    stage('Continuous Integration') {
      steps {
        parallel 'phplint' : {
            sh 'vendor/bin/parallel-lint --exclude vendor .'
          },
          'phploc' : {
            sh 'vendor/bin/phploc --count-tests --log-csv build/logs/phploc.csv --log-xml build/logs/phploc.xml --exclude=vendor .'
          },
          'phpunit' : {
            sh 'vendor/bin/phpunit --configuration phpunit.xml --coverage-html build/coverage --coverage-clover build/logs/clover.xml --coverage-crap4j build/logs/crap4j.xml --log-junit build/logs/junit.xml'
          },
          'phpdepend' : {
            sh 'vendor/bin/pdepend --jdepend-xml=build/logs/jdepend.xml --jdepend-chart=build/pdepend/dependencies.svg --overview-pyramid=build/pdepend/overview-pyramid.svg --ignore=vendor .'
          },
          'phpmd' : {
            sh 'vendor/bin/phpmd . xml phpmd.xml --reportfile build/logs/pmd.xml || true'
          },
          'phpdox' : {
            sh 'vendor/bin/phpdox'
          },
          'phpcs' : {
            sh 'vendor/bin/phpcs --report=checkstyle --report-file=build/logs/checkstyle.xml'
          },
          'phpcpd' : {
            sh 'vendor/bin/phpcpd --exclude vendor --log-pmd build/logs/pmd-cpd.xml .'
          }
      }
    }

    stage('Publishing') {
      steps {
        parallel 'phploc' : {
          echo '@TODO phploc'
        },
        'phpunit' : {
          step([$class: 'XUnitPublisher', testTimeMargin: '3000', thresholdMode: 1, thresholds: [[$class: 'FailedThreshold', failureNewThreshold: '', failureThreshold: '', unstableNewThreshold: '', unstableThreshold: ''], [$class: 'SkippedThreshold', failureNewThreshold: '', failureThreshold: '', unstableNewThreshold: '', unstableThreshold: '']], tools: [[$class: 'JUnitType', deleteOutputFiles: true, failIfNotNew: false, pattern: 'build/logs/junit.xml', skipNoTestFiles: false, stopProcessingIfError: true]]])
          step([$class: 'CloverPublisher',cloverReportDir: 'build/logs',cloverReportFileName: 'clover.xml',healthyTarget: [methodCoverage: 70, conditionalCoverage: 80, statementCoverage: 80],unhealthyTarget: [methodCoverage: 50, conditionalCoverage: 60, statementCoverage: 60],failingTarget: [methodCoverage: 30, conditionalCoverage: 40, statementCoverage: 40]])
          publishHTML(target: [reportName: 'PHPUnit Reports',reportDir: 'build/pdepend',reportFiles: '',keepAll: true])
        },
        'phpdepend' : {
          publishHTML(target: [reportName: 'PDepend Reports',reportDir: 'build/pdepend',reportFiles: '',keepAll: true])
        },
        'phpmd' : {
          step([$class: 'PmdPublisher', canComputeNew: false, pattern: 'build/logs/pmd.xml'])
        },
        'phpdox' : {
          publishHTML(target: [reportName: 'PHPDoc Reports',reportDir: 'build/phpdox/',reportFiles: '',keepAll: true])
        },
        'phpcs' : {
          step([$class: 'CheckStylePublisher', pattern: 'build/logs/checkstyle.xml'])
        },
        'phpcpd' : {
          step([$class: 'DryPublisher', canComputeNew: false, defaultEncoding: '', healthy: '', pattern: 'build/logs/pmd-cpd.xml', unHealthy: ''])
        }
      }
    }

  }
}
