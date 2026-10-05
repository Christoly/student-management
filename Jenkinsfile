pipeline {
    agent any

    tools {
        jdk 'JAVA_HOME'
        maven 'M2_HOME'
    }

    triggers {
        pollSCM('H/5 * * * *')
    }

    environment {
       IMAGE_NAME = 'christoly/student-management'
       IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {

        stage('Commit') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Christoly/student-management.git'

                sh '''
                    echo "Dernier commit :"
                    git log -1 --pretty=format:"Hash: %h%nAuteur: %an%nMessage: %s"
                '''
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
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG -t $IMAGE_NAME:latest .'
            }
       }

       stage('Docker Push') {
           steps {
               withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials',
                                                 usernameVariable: 'DH_USER',
                                                 passwordVariable: 'DH_TOKEN')]) {
                   sh 'echo "$DH_TOKEN" | docker login -u "$DH_USER" --password-stdin'
                   sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
                   sh 'docker push $IMAGE_NAME:latest'
               }
           }
       }
    }

    post {
        always {
            sh 'docker logout || true'
        }

        success {
            archiveArtifacts artifacts: 'target/*.jar',
                             fingerprint: true

            echo 'Pipeline terminé avec succès : tests réussis et JAR archivé.'
        }

        failure {
            echo 'Pipeline en échec : consultez les résultats des tests et les logs.'
        }
    }
}
