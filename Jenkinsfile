pipeline {
    agent any
    stages {
        stage('Build Artefact') {
            steps {
               sh "mvn clean package -DskipTest=true"
            }
        }
        stage('Lancer les Tests') {
            steps {
               sh "mvn test"
            }
        }
        stage('Build & push img') {
            steps {
               withDockerRegistry(credentialsId:"dockerhub", url:""){
                    sh "docker build -t ghostcodebaba/demoboots:$BUILD_NUMBER ."
                    sh "docker push ghostcodebaba/demoboots:$BUILD_NUMBER"
               }
              
            }
        }
        stage('Start container') {
            steps {
               sh "docker stop demoboots||true"
               sh "docker rm demoboots||true"
               sh "docker run -d --name demoboots -p 8180:8180 ghostcodebaba/demoboots:$BUILD_NUMBER"
            }
        }
    }
}