pipeline {
    tools{
        nodejs "nodejs"
    }
    agent any
    stages {
        stage('Tests') {
            steps {
                script {
                   docker.image('node:10-stretch').inside { c ->
                        echo 'Building..'
                        sh 'npm install'
                        echo 'Testing..'
                        sh 'npm test'
                        sh "docker logs ${c.id}"
                   }
                }
            }
        }
        stage('Build and push docker image') {
            steps {
                script {
                    echo 'build'
                    def dockerImage = docker.build("aymen/node-demo:master")
                    docker.withRegistry('https://registry.hub.docker.com', 'demo-docker') {
                        dockerImage.push('master')
                    }
                }
            }
        }
        stage('Deploy to remote docker host') {
            environment {
                DOCKER_HOST_CREDENTIALS = credentials('demo-docker')
            }
            steps {
                script {
                    sh 'docker login -u $DOCKER_HOST_CREDENTIALS_USR -p $DOCKER_HOST_CREDENTIALS_PSW'
                    sh 'docker pull aymen/node-demo:master'
                    sh 'docker stop node-demo'
                    sh 'docker rm node-demo'
                    sh 'docker rmi aymen/node-demo:current'
                    sh 'docker tag aymen/node-demo:master aymen/node-demo:current'
                    sh 'docker run -d --name node-demo -p 80:3000 aymen/node-demo:current'
                    echo "deploy"
                }
            }
        }
    }
}
