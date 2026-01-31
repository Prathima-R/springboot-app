pipeline {
    agent any
    tools {
        jdk 'java-17'
        maven 'maven'
    }
    stages {
        stage ("GITCHECKOUT"){
            steps {
                git branch: 'main', url:'https://github.com/Prathima-R/springboot-app.git'
            }
        }
        stage ("compile") {
            steps {
                sh 'mvn compile'
            }
        }
        stage ("build") {
            steps {
                sh 'mvn clean package'
            }
        }
       stage ("sonarqube-scan") {
        steps {
          sh '''
             mvn sonar:sonar \
             -Dsonar.projectKey=java \
             -Dsonar.host.url=http://65.2.126.97:9000 \
            -Dsonar.login=afb35d39d9cf734f2cbfe5842188de8d38813ba1
     '''

        }
       }
        stage ("IMAGE BUILD") {
            steps {
                sh 'docker build -t prathima2025/springboot-app:${GIT_COMMIT} .'
            }
        }
        stage ("PUSH IMAGE") {
            steps {
                script {
                    script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                        sh "docker push prathima2025/springboot-app:${GIT_COMMIT}"
                    }
                }
                stage ("DEPLOYMENT") {
                    steps {
                        withKubeConfig(credentialsId: 'kubecredID') {
                    sh '''
                        kubectl apply -f Deployment.yml
                        kubectl apply -f service.yml
                    '''
                    }   
                }

            }
        }
    }
    
}
    }
}

