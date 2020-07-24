#!/usr/bin/env groovy

pipeline {
    agent any

    triggers {
        pollSCM('*/15 * * * *')
    }

    options { disableConcurrentBuilds() }

    stages {
        stage('Permissions') {
            steps {
                sh 'chmod 775 *'
            }
        }
		
		stage('Cleanup') {
            steps {
                sh './gradlew --no-daemon clean'
            }
        }


		
		stage('Test') {
            steps {
                sh './gradlew --no-daemon check'
            }
            post {
                always {
                    junit 'build/test-results/test/*.xml'
                }
            }
        }

        stage('Build') {
            steps {
                sh './gradlew --no-daemon build'
            }
        }

        stage('Update Docker UAT image') {
            when { branch "master" }
            steps {
                sh '''
					docker login -u "msarvala" -p "malli@1234"
                    docker build --no-cache -t person .
                    docker tag person:latest msarvala/person:latest
                    docker push msarvala/person:latest
					docker rmi person:latest
                '''
            }
        }

        stage('Update UAT container') {
            when { branch "master" }
            steps {
                sh '''
					docker login -u "msarvala" -p "malli@1234"
                    docker pull msarvala/person:latest
                    docker stop person
					docker rm person
					docker run -p 9090:9090 --name person -t -d msarvala/person
					docker rmi -f $(docker images -q --filter dangling=true)
                '''
            }
        }

        stage('Release Docker image') {

            steps {
                sh '''
					docker login -u "msarvala" -p "malli@1234"
                    docker build --no-cache -t person .
                    docker tag person:latest msarvala/person:latest
                    docker push msarvala/person:latest
					docker rmi $(docker images -f "dangling=true" -q)
               '''
            }
        }
    }
}
