pipeline{
    agent{
        docker{
            image 'harshalbh/maven-harshalb-docker-agent:v1'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    stages{
        stage('Checkout') {
            steps{
                sh 'echo passed'
            }
        }
        stage('Build and Test') {
            steps{
                sh 'ls -ltr'
                sh 'cd spring-boot-app && mvn clean package'
            }
        }
        stage('Static Code Analysis') {
            environment{
                SONAR_URL = 'http://3.88.14.47:9000'
            }
            steps{
                withCredentials([string(credentialsId: 'SonarQube', variable: 'SONAR_AUTH_TOKEN')]){
                    sh 'cd spring-boot-app && sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
                }
            }
        }
        stage('Build and Push Docker Image') {
            environment {
                DOCKER_IMAGE = "harshalbh/ultimate-cicd:${BUILD_NUMBER}"
                REGISTRY_CREDENTIALS = credentials('docker-cred')
            }
            steps{
                script {
                    sh 'cd spring-boot-app && docker build -t ${DOCKER_IMAGE} .'
                    def dockerImage = docker.image("${DOCKER_IMAGE}")
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-cred') {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = 'UltimateCICD'
                GIT_USER_NAME = 'harshalbhope'
            }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                    git config user.email "harshalbhope67@gmail.com"
                    git config user.name "Harshal Bhope"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" spring-boot-app-manifests/deployment.yml
                    git add spring-boot-app-manifests/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                }
            }
        }
    }
}