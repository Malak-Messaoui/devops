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

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh 'kubectl apply -f k8s/'
                    sh 'kubectl rollout status deployment/spring-deployment'
                }
            }
        }
    }
}
