pipeline {
    agent any

    tools {
        jdk 'jdk-17'
        maven 'maven3'
    }

    environment {
        DOCKER_IMAGE = "aslamshaikh235/expenses-app"
        DOCKER_TAG = "latest"
    }

    stages {
        
        stage("Checkout code") {
            steps {
                git branch: 'main', url:"https://github.com/aslamshaikh235/Expenses-Tracker-WebApp-DockerProj-2.git"
            }
        }

        stage('Build with Mave') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=expenses-app \
                    -Dsonar.organization=mvn-app-org \
                    -Dsonar.host.url=https://sonarcloud.io \
                    -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$DOCKER_TAG .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable:'DOCKER_USER', passwordVariable:'DOCKER_PASS')]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $DOCKER_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                echo "Deploying to EKS..."

                # Apply deployment
                kubectl apply -f k8s/deployment.yml

                # Apply service
                kubectl apply -f k8s/service.yml

                # Verify
                kubectl get pods
                kubectl get svc
                '''
            }
        }
    } 

    post {
        success {
            echo  "Pipeline SUCCESS 🚀"
        }
        failure {
            echo "Pipeline FAILED ❌"
        }
    }
}