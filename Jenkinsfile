pipeline {
    agent any

    tools {
        jdk 'JDK21'
        maven 'Maven3'
    }

    environment {
        PATH = "C:\\Program Files\\Docker\\Docker\\resources\\bin;${env.PATH}"
        SONARQUBE_SERVER = 'SonarQubeServer'
        SONAR_CREDENTIALS_ID = 'SONAR_TOKEN'
        DOCKERHUB_CREDENTIALS_ID = 'Docker_ID'
        DOCKERHUB_REPO = 'khaledmarai/bmi_test_3013'
        DOCKER_IMAGE_TAG = 'latest'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                        url: 'https://github.com/Khaleddamascene/RTL_BMI_Solution_Instructions.git'
            }
        }

        stage('Build') {
            steps {
                bat 'mvn -B clean verify'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([usernamePassword(
                        credentialsId: "${SONAR_CREDENTIALS_ID}",
                        usernameVariable: 'SONAR_USER',
                        passwordVariable: 'SONAR_TOKEN'
                )]) {
                    withSonarQubeEnv("${SONARQUBE_SERVER}") {
                        bat 'mvn -B org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.host.url=%SONAR_HOST_URL% -Dsonar.projectKey=BMI_Cal -Dsonar.projectName=BMI_Cal -Dsonar.token=%SONAR_TOKEN%'
                    }
                }
            }
        }

        stage('Check Dockerfile') {
            steps {
                bat 'if not exist Dockerfile (echo Dockerfile is missing & exit /b 1)'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    docker.withRegistry(
                            'https://index.docker.io/v1/',
                            DOCKERHUB_CREDENTIALS_ID
                    ) {
                        docker.image(
                                "${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}"
                        ).push()
                    }
                }
            }
        }
    }
}