pipeline {
    agent any
    environment {
        registry = 'ghostcodebaba/demo-devsecops'
        registryCredential = 'DockerHub'
        kubernetesCredential = 'kubernetes-config'
    }

    stages {
        stage('Build Artifact') {
            // when { expression { false } }
            steps {
                sh 'mvn clean package -DskipTest=true'
                archive 'target/*.jar' //so that they can be downloaded later
            }
        }

        stage('Unit Tests - JUnit and Jacoco') {
            steps {
                sh 'mvn test -Dgroups=unitaires'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                    jacoco execPattern: 'target/jacoco.exec'
                }
            }
        }

        stage('Service - IntegrationTest') {
            steps {
                sh 'mvn test -Dgroups=integrations'
            }
        }

        stage('Web - IntegrationTest') {
            steps {
                sh 'mvn test -Dgroups=web'
            }
        }

        stage('Mutation Tests - PIT') {
            steps {
                sh 'mvn org.pitest:pitest-maven:mutationCoverage'
            }
            post {
                always {
                    pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
                }
            }
        }

        stage('Docker Build and Push') {
            when { expression { false } }
            steps {
                withDockerRegistry([credentialsId: registryCredential, url: '']) {
                    sh 'printenv'
                    sh 'docker build -t $registry:$BUILD_NUMBER .'
                    sh 'docker push $registry:$BUILD_NUMBER'
                }
            }
        }

        stage('Remove Unused docker image') {
            when { expression { false } }
            steps {
                sh "docker rmi $registry:$BUILD_NUMBER"
            }
        }

        stage('Kubernetes Deployment - DEV') {
            when { expression { false } }
            steps {
                withKubeConfig([credentialsId: kubernetesCredential]) {
                    sh "sed -i 's#replace#${registry}:${BUILD_NUMBER}#g' k8s_deployment_service.yaml"
                    sh 'kubectl apply -f k8s_deployment_service.yaml'
                }
            }
        }
    }
}
