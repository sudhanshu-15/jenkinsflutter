# A Docker image with Jenkins and Flutter installed

## Instructions:
This image is based on jenkins/jenkins:lts, all basic instructions from https://github.com/jenkinsci/docker/blob/master/README.md are applicable to this image. Flutter from Flutter beta channel is already installed on this image. This image has PATH set to be able to access `flutter` and  `dart-sdk` tools from the terminal.

Detailed instruction of setting up a Jenkins pipeline using this image is available in these posts:
* Medium 1
* Medium 2

A sample Jenkinsfile is available here:
```
pipeline {
    agent any
    stages {
        stage ('Checkout') {
            steps {
                checkout scm
            }
        }
        stage ('Download lcov converter') {
            steps {
                sh "curl -O https://raw.githubusercontent.com/eriwen/lcov-to-cobertura-xml/master/lcov_cobertura/lcov_cobertura.py"
            }
        }
        stage ('Flutter Doctor') {
            steps {
                sh "flutter doctor"
            }
        }
        stage('Test') {
            steps {
                sh "flutter test --coverage"
            }
            post {
                always {
                    sh "python3 lcov_cobertura.py coverage/lcov.info --output coverage/coverage.xml"
                    step([$class: 'CoberturaPublisher', coberturaReportFile: 'coverage/coverage.xml'])
                }
            }
        }
        stage('Run Analyzer') {
            steps {
                sh "dartanalyzer --options analysis_options.yaml ."
            }
        }
    }
}
```