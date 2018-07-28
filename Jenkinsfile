def userName = "seanmaber"

def gitCommit() {
    sh "git rev-parse HEAD > GIT_COMMIT"
    def gitCommit = readFile('GIT_COMMIT').trim()
    sh "rm -f GIT_COMMIT"
    return gitCommit
}

pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh "docker build -t mesosphere/software-architecture:${userName}-${gitCommit()} ."
            }
        }
        stage('Publish') {
            steps {
                withCredentials(
                    [[
                        $class: 'UsernamePasswordMultiBinding',
                        credentialsId: 'docker-hub-credentials',
                        passwordVariable: 'DOCKERHUB_PASSWORD',
                        usernameVariable: 'DOCKERHUB_USERNAME'
                    ]]
                ) {
                    sh "docker login -u '${env.DOCKERHUB_USERNAME}' -p '${env.DOCKERHUB_PASSWORD}' -e demo@mesosphere.com"
                    sh "docker push mesosphere/software-architecture:${userName}-${gitCommit()}"
                }
            }
        }
        stage('Deploy') {
            steps {
                marathon(
                    url: 'http://marathon.mesos:8080',
                    forceUpdate: false,
                    credentialsId: 'dcos-token',
                    filename: 'marathon.json',
                    appId: 'nginx-${userName}',
                    docker: "mesosphere/software-architecture:${userName}-${gitCommit()}".toString()
                )
            }
        }
    }
}
