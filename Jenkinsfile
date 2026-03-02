pipeline {
    agent any
    
    stages {
        stage('GIT') {
            steps {
                git branch: 'main',
                    changelog: false,
                    credentialsId: 'jenkins.github',
                    url: 'https://github.com/Malak-Messaoui/devops.git'
            }
        }
        
        stage('MAVEN Build') {
            steps {
                sh 'mvn clean compile package -DskipTests'
            }
        }
        
        stage('SONARQUBE') {
            environment {
                SONAR_HOST_URL = 'http://192.168.50.4:9000/'
                SONAR_AUTH_TOKEN = credentials('sonarqube')
            }
            steps {
                sh 'mvn sonar:sonar -Dsonar.projectKey=devops_git -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.token=$SONAR_AUTH_TOKEN'
            }
        }

        stage('Docker Build') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker build -t malakmessaoui/spring-k8s-app:2.0 .
                        docker push malakmessaoui/spring-k8s-app:2.0
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG
                        kubectl get nodes
                        kubectl apply -f k8s/
                    '''
                }
            }
        }
    }
}
