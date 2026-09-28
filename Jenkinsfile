pipeline {
    agent any

    tools {
        jdk 'JAVA_HOME'
        maven 'M2_HOME'
    }

    triggers {
        pollSCM('H/5 * * * *')
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
    }

    post {
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