pipeline {
    agent any

    stages {
        stage('SCM') {
            steps {
                checkout scm
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    def mvnHome = tool name: 'Default Maven', type: 'maven'
                    withSonarQubeEnv('ProbandoSonar') {
                        sh "${mvnHome}/bin/mvn clean verify sonar:sonar -Dsonar.login=squ_b2ff83cfbe9e664fedfa80737a981c65090d2ebe -Dsonar.projectKey=Projecto_jenkins -Dsonar.projectName='Projecto_jenkins'"
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed'
        }
    }
}
