pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Build') {
      agent any
      environment {
        x = '10'
      }
      steps {
        sh "docker build -t mesosphere/software-architecture:${userName}-${gitCommit()} ."
      }
    }
    stage('Publish') {
      steps {
        withCredentials(bindings: [[
                                  $class: 'UsernamePasswordMultiBinding',
                                  credentialsId: 'docker-hub-credentials',
                                  passwordVariable: 'DOCKERHUB_PASSWORD',
                                  usernameVariable: 'DOCKERHUB_USERNAME'
                              ]]) {
            sh "docker login -u '${env.DOCKERHUB_USERNAME}' -p '${env.DOCKERHUB_PASSWORD}' -e demo@mesosphere.com"
            sh "docker push mesosphere/software-architecture:${userName}-${gitCommit()}"
          }

        }
      }
      stage('Deploy') {
        steps {
          marathon(url: 'http://marathon.mesos:8080', credentialsId: 'dcos-token', filename: 'marathon.json', appId: 'nginx-${userName}', docker: "mesosphere/software-architecture:${userName}-${gitCommit()}".toString())
        }
      }
    }
  }