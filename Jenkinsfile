pipeline {
    agent any
    stages {
        stage('SCM Checkout'){
            steps {
            git branch: 'main', url: 'https://github.com/naresh26git/microservices.git'
            sh 'ls'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                parallel (
                    'node application': {
                        script {
                            dir('cart-microservice-nodejs') {
                                def scannerHome = tool 'sonarscanner4'
                                 withSonarQubeEnv('sonar-pro') {
                                     sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=cart-nodejs"
                                 }
                            }
                            dir('ui-web-app-reactjs') {
                                def scannerHome = tool 'sonarscanner4';
                                withSonarQubeEnv('sonar-pro') {
                                    sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=ui-reactjs"
                                }
                            }
                        }
                    },
                    'spring boot application': {
                        script {
                            dir('offers-microservice-spring-boot') {
                                def mvn = tool 'maven3';
                                withSonarQubeEnv('sonar-pro') {
                                    sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=offers-spring-boot -Dsonar.projectName=offers-spring-boot"
                                }
                            }
                            dir('shoes-microservice-spring-boot') {
                                def mvn = tool 'maven3';
                                withSonarQubeEnv('sonar-pro') {
                                    sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=shoe-spring-boot -Dsonar.projectName=shoes-spring-boot"
                                }
                            }
                            dir('zuul-api-gateway') {
                                def mvn = tool 'maven3';
                                withSonarQubeEnv('sonar-pro') {
                                    sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=zuul-api -Dsonar.projectName=zuul-api"
                                }
                            }
                        }
                    },
                    'python app': {
                        script{
                            dir('wishlist-microservice-python') {
                                def scannerHome = tool 'sonarscanner4';
                                withSonarQubeEnv('sonar-pro') {
                                    sh """/var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/sonarscanner4/bin/sonar-scanner \
                                    -D sonar.projectVersion=1.0-SNAPSHOT \
                                    -D sonar.sources=. \
                                    -D sonar.login=admin \
                                    -D sonar.password=admin123 \
                                    -D sonar.projectKey=project \
                                    -D sonar.projectName=wishlist-py \
                                    -D sonar.inclusions=index.py \
                                    -D sonar.sourceEncoding=UTF-8 \
                                    -D sonar.language=python \
                                    -D sonar.host.url=http://13.232.29.132:9000/"""
                                }
                            }
                        }
                    }
                )
            }
        }
        stage ('Build Docker Image and push'){
            steps {
                parallel (
                    'docker login': {
                        withCredentials([string(credentialsId: 'dockerPass', variable: 'dockerPassword')]) {
                            sh "docker login -u vivekanand1997 -p ${dockerPassword}"
                        }
                    },
                    'ui-web-app-reactjs': {
                        dir('ui-web-app-reactjs'){
                            sh """
                            docker build -t vivekanand1997/ui:v1 .
                            docker push vivekanand1997/ui:v1
                            docker rmi vivekanand1997/ui:v1
                            """
                        }
                    },
                    'zuul-api-gateway' : {
                        dir('zuul-api-gateway'){
                            sh """
                            docker build -t vivekanand1997/api:v1 .
                            docker push vivekanand1997/api:v1
                            docker rmi vivekanand1997/api:v1
                            """
                        }
                    },
                    'offers-microservice-spring-boot': {
                        dir('offers-microservice-spring-boot'){
                            sh """
                            docker build -t vivekanand1997/spring:v1 .
                            docker push vivekanand1997/spring:v1
                            docker rmi vivekanand1997/spring:v1
                            """
                        }
                    },
                    'shoes-microservice-spring-boot': {
                        dir('shoes-microservice-spring-boot'){
                            sh """
                            docker build -t vivekanand1997/spring:v2 .
                            docker push vivekanand1997/spring:v2
                            docker rmi vivekanand1997/spring:v2
                            """
                        }
                    },
                    'cart-microservice-nodejs': {
                        dir('cart-microservice-nodejs'){
                            sh """
                            docker build -t vivekanand1997/ui:v2 .
                            docker push vivekanand1997/ui:v2
                            docker rmi vivekanand1997/ui:v2
                            """
                        }
                    },
                    'wishlist-microservice-python': {
                        dir('wishlist-microservice-python'){
                            sh """
                            docker build -t vivekanand1997/python:v1 .
                            docker push vivekanand1997/python:v1
                            docker rmi vivekanand1997/python:v1
                            """
                        }
                    }
                )
            }
        }
        stage ('Deploy on k8s'){
            steps{
                parallel (
                    'deploy on k8s': {
                        script {
                            withKubeCredentials(kubectlCredentials: [[ credentialsId: 'kubernetes', namespace: 'ms' ]]) {
                                sh 'kubectl get ns' 
                                sh 'kubectl apply -f kubernetes/yamlfile'
                            }
                        }
                    }
                )
            }
        }
    }
}
