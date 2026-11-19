pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'M2_HOME'
    }

    environment {
        IMAGE_NAME = 'medaligdira/student-management'
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
    }

    triggers {
        pollSCM('H/5 * * * *')
    }

    stages {
        stage('Commit') {
            steps {
                sh 'git log -1 --pretty=format:"%h %an %s"'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test unitaire') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh "sudo docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_PASS'
                )]) {
                    sh 'echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin'
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
        success {
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            echo "Pipeline succeeded: image ${IMAGE_NAME}:${IMAGE_TAG} pushed and jar archived."
        }
        failure {
            echo 'Pipeline failed: check the Console Output for details.'
        }
    }
}
