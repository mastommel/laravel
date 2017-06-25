#!groovy

pipeline {
  agent any

  stages {
    stage('Prepare') {
        sh 'mkdir -p build/{coverage,html,logs,pdepend,phpdox}'
        sh 'composer install'
    }
    stage('php-lint') {
        sh 'vendor/bin/parallel-lint --exclude vendor .'
    }
    stage('php-loc') {
        sh 'vendor/bin/phploc --count-tests --log-csv build/logs/phploc.csv --log-xml build/logs/phploc.xml --exclude=vendor .'
    }
    stage('php-unit') {
        sh 'vendor/bin/phpunit --configuration phpunit.xml --coverage-html build/coverage --coverage-clover build/logs/clover.xml --coverage-crap4j build/logs/crap4j.xml --log-junit build/logs/junit.xml'
    }
    stage('php-depend') {
        sh 'vendor/bin/pdepend --jdepend-xml=build/logs/jdepend.xml --jdepend-chart=build/pdepend/dependencies.svg --overview-pyramid=build/pdepend/overview-pyramid.svg --ignore=vendor .'
    }
    stage('php-md') {
        sh 'vendor/bin/phpmd . text phpmd.xml'
    }
    stage('php-dox') {
        sh 'vendor/bin/phpdox'
    }
    stage('php-cs') {
        sh 'vendor/bin/phpcs'
    }
    stage('php-cpd') {
        sh 'vendor/bin/phpcpd .'
    }
    stage('Reporting') {
        step([$class: 'hudson.plugins.checkstyle.CheckStylePublisher', pattern: 'build/logs/checkstyle.xml'])
        step([$class: 'PmdPublisher', canComputeNew: false, pattern: 'build/logs/pmd.xml'])
        step([$class: 'DryPublisher', canComputeNew: false, defaultEncoding: '', healthy: '', pattern: 'build/logs/pmd-cpd.xml', unHealthy: ''])
        step([$class: 'JavadocArchiver', javadocDir: 'build/api', keepAll: false])
        step([$class: 'XUnitPublisher', testTimeMargin: '3000', thresholdMode: 1, thresholds: [[$class: 'FailedThreshold', failureNewThreshold: '', failureThreshold: '', unstableNewThreshold: '', unstableThreshold: ''], [$class: 'SkippedThreshold', failureNewThreshold: '', failureThreshold: '', unstableNewThreshold: '', unstableThreshold: '']], tools: [[$class: 'JUnitType', deleteOutputFiles: true, failIfNotNew: false, pattern: 'build/logs/junit.xml', skipNoTestFiles: false, stopProcessingIfError: true]]])
    }
}
