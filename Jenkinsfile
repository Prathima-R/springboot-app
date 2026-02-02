pipeline {
    agent any

    tools {
        jdk 'java-17'
        maven 'maven'
    }

    stages {

        stage('GIT CHECKOUT') {
            steps {
                git branch: 'main', url: 'https://github.com/Prathima-R/springboot-app.git'
            }
        }

        stage('COMPILE') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('BUILD') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SONARQUBE SCAN') {
            steps {
                sh '''
                mvn sonar:sonar \
                -Dsonar.projectKey=java \
                -Dsonar.host.url=http://65.2.126.97:9000 \
                -Dsonar.login=afb35d39d9cf734f2cbfe5842188de8d38813ba1
                '''
            }
        }

        stage('IMAGE BUILD') {
            steps {
                sh 'docker build -t prathima2025/springboot-app:${GIT_COMMIT} .'
            }
        }

        stage('PUSH IMAGE') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-credentials',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    sh 'docker push prathima2025/springboot-app:${GIT_COMMIT}'
                }
            }
        }

        stage('DEPLOYMENT') {
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

post {
        success {
            emailext (
                subject: "✅ SUCCESS: ${JOB_NAME} #${BUILD_NUMBER}",
                body: """
Hi Team,

Good news 🎉

Pipeline completed successfully.

Job Name   : ${JOB_NAME}
Build No   : ${BUILD_NUMBER}
Status     : SUCCESS
Build URL  : ${BUILD_URL}

Regards,
Jenkins
""",
                to: "prathimaprati504@gmail.com"
            )
        }

        failure {
            emailext (
                subject: "❌ FAILURE: ${JOB_NAME} #${BUILD_NUMBER}",
                body: """
Hi Team,

Pipeline has FAILED ❌

Job Name   : ${JOB_NAME}
Build No   : ${BUILD_NUMBER}
Status     : FAILURE
Build URL  : ${BUILD_URL}

Please check logs for details.

Regards,
Jenkins
""",
                to: "prathimaprati504@gmail.com"
            )
        }
    }
}
