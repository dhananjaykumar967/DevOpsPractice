pipeline {
    agent any

    tools {
        maven 'Maven3'
        jdk 'JDK21'
    }

    environment {
        REGISTRY = 'dhananjaykumar967'
        DOCKER_IMAGE = 'maven-hello-world'
        DOCKER_TAG = "${BUILD_NUMBER}"
        APP_HOST = '172.31.13.55'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
                sh 'cp webapp/target/webapp.war webapp.war'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $REGISTRY/$DOCKER_IMAGE:$DOCKER_TAG -t $REGISTRY/$DOCKER_IMAGE:latest .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh 'docker push $REGISTRY/$DOCKER_IMAGE:$DOCKER_TAG'
                    sh 'docker push $REGISTRY/$DOCKER_IMAGE:latest'
                }
            }
        }

        stage('Deploy') {
            steps {
                sh 'ANSIBLE_HOST_KEY_CHECKING=False ansible all -i "$APP_HOST," -u ec2-user --private-key /var/lib/jenkins/.ssh/app_key.pem -m shell -a "docker pull $REGISTRY/$DOCKER_IMAGE:$DOCKER_TAG && docker rm -f maven-hello-world 2>/dev/null; docker run -d --name maven-hello-world --restart always -p 80:8080 $REGISTRY/$DOCKER_IMAGE:$DOCKER_TAG"'
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded! App should be live at http://13.233.155.131/'
        }
        failure {
            echo 'Pipeline failed. Check the stage logs above.'
        }
        always {
            sh 'docker logout || true'
        }
    }
}
