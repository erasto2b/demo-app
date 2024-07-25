pipeline {
    agent any

    environment {
        GIT_CREDENTIALS = '80fb7680-e9da-48aa-80b6-d96387fbafec' // ID de credenciales de Git en Jenkins
        DOCKER_HUB_CREDENTIALS = 'docker-hub-credentials' // ID de credenciales de Docker Hub en Jenkins
        SONARQUBE_SERVER = 'SonarQube_Localhost' // Nombre del servidor SonarQube configurado en Jenkins
        SONARQUBE_TOKEN = 'squ_b2ff83cfbe9e664fedfa80737a981c65090d2ebe' // Token de autenticación para SonarQube
        MAVEN_HOME = tool 'Maven' // Nombre de la herramienta Maven configurada en Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/seiler18/AppManageEvents', branch: 'main', credentialsId: GIT_CREDENTIALS
            }
        }
        stage('Build and Package with Maven') {
            steps {
                script {
                    def mvnHome = tool 'Maven'
                    withMaven(maven: mvnHome, mavenSettingsConfig: 'your-maven-settings') {
                        sh './mvnw clean package -DskipTests'
                    }
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube_Localhost') {
                        sh './mvnw sonar:sonar -Dsonar.host.url=http://localhost:9000 -Dsonar.login=${SONARQUBE_TOKEN}'
                    }
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_HUB_CREDENTIALS) {
                        def dockerImage = docker.build("seiler18/mascachicles:AppFinalRelease-${env.BUILD_NUMBER}", ".")
                        dockerImage.push()
                    }
                }
            }
        }
        // Si deseas desplegar la aplicación, puedes habilitar esta etapa
        // stage('Deploy') {
        //     steps {
        //         sh './mvnw spring-boot:run'
        //     }
        // }
    }

    post {
        always {
            echo 'Pipeline completed'
        }
        cleanup {
            echo 'Cleaning up old Docker images'
            sh '''
                docker images --filter "reference=seiler18/mascachicles" --format "{{.ID}}" | tail -n +3 | xargs -r docker rmi -f
                docker images --filter "reference=registry.hub.docker.com/seiler18/mascachicles" --format "{{.ID}}" | tail -n +3 | xargs -r docker rmi -f
            '''
        }
    }
}
