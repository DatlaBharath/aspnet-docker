pipeline {
    agent any
    tools {
        dotnet 'dotNet'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/DatlaBharath/aspnet-docker'
            }
        }
        stage('Build') {
            steps {
                sh 'dotnet build --configuration Release --no-restore --no-build --no-dependencies'
            }
        }
        stage('Docker Build and Push') {
            steps {
                script {
                    def repoName = 'aspnet-docker'
                    def tag = "ratneshpuskar/${repoName.toLowerCase()}:${env.BUILD_NUMBER}"
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                        sh 'echo "$DOCKER_HUB_PASSWORD" | docker login -u "$DOCKER_HUB_USERNAME" --password-stdin'
                        sh "docker build -t ${tag} ."
                        sh "docker push ${tag}"
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deploymentYaml = '''
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: aspnet-docker-deployment
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: aspnet-docker
                      template:
                        metadata:
                          labels:
                            app: aspnet-docker
                        spec:
                          containers:
                          - name: aspnet-docker
                            image: ratneshpuskar/aspnet-docker:${BUILD_NUMBER}
                            ports:
                            - containerPort: 80
                    '''

                    def serviceYaml = '''
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: aspnet-docker-service
                    spec:
                      type: NodePort
                      selector:
                        app: aspnet-docker
                      ports:
                      - protocol: TCP
                        port: 80
                        nodePort: 30007
                    '''

                    writeFile file: 'deployment.yaml', text: deploymentYaml
                    writeFile file: 'service.yaml', text: serviceYaml

                    sh '''
                      ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@3.6.238.137 "kubectl apply -f -" < deployment.yaml
                      ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@3.6.238.137 "kubectl apply -f -" < service.yaml
                    '''
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment was successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}
