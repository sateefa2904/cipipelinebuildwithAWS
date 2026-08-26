pipeline {
    agent any

    options {
        timestamps()
    }

    environment {
        IMAGE_REPO = 'sateefa2904/team-skeleton'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    // Build app without running tests here; tests run in the next stage.
                    if (isUnix()) {
                        sh 'mvn -B clean package -DskipTests'
                    } else {
                        bat 'mvn -B clean package -DskipTests'
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'mvn -B test'
                    } else {
                        bat 'mvn -B test'
                    }
                }
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def image = "${IMAGE_REPO}:${IMAGE_TAG}"
                    if (isUnix()) {
                        sh "docker build -t ${image} ."
                    } else {
                        bat "docker build -t ${image} ."
                    }
                }
            }
        }

        stage('Push Docker Image') {
            when {
                branch 'main'
            }
            steps {
                script {
                    def image = "${IMAGE_REPO}:${IMAGE_TAG}"
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        if (isUnix()) {
                            sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                            sh "docker push ${image}"
                            sh 'docker logout'
                        } else {
                            bat '@echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin'
                            bat "docker push ${image}"
                            bat 'docker logout'
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Build ${BUILD_NUMBER} completed successfully."
        }
        failure {
            echo "Build ${BUILD_NUMBER} failed. Check console output for details."
        }
    }
}
