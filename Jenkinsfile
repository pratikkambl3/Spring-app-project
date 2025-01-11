pipeline{
  agent {
   docker {
    image 'abhishekf5/maven-abhishek-docker-agent:v1'
    args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
    }
  }
    stages{
        stage('Git-Checkout'){
            steps{
                sh 'echo "checkout done"'
            }
        }
        stage('Build'){
            steps{
                sh 'mvn clean package'
            }
        }
        stage('static code analysis'){
            environment {
                SONAR_URL = "http://35.154.251.84:9000"
            }
            steps{
                withCredentials([string(credentialsId: 'sonar', variable: 'sonar-token')]) {
                    sh 'mvn sonar:sonar -Dsonar.login=$sonar-token -Dsonar.host.url=${SONAR_URL}'
                }           

            }
        }
        stage('Image Build and push to Dockerhub'){
             environment {
                DOCKER_IMAGE = "pratikkambl3/spring-app:${BUILD_NUMBER}"
                // DOCKERFILE_LOCATION = "java-maven-sonar-argocd-helm-k8s/spring-boot-app/Dockerfile"
                REGISTRY_CREDENTIALS = credentials('Docker-login')
            }     
            steps{
                script {
                docker build -t ${DOCKER_IMAGE} .
                def dockerImage = docker.image("${DOCKER_IMAGE}")
                docker.withRegistry('https://index.docker.io/v1/', "Docker-login") {
                dockerImage.push()
             }
            
            }
        }
        stage('Update Deployment file'){
            steps{
                environment {
            GIT_REPO_NAME = "Spring-app-project"
            GIT_USER_NAME = "pratikkambl3"
        }
        steps {
            withCredentials([string(credentialsId: 'Github-Token', variable: 'Github-Token')]) {
                sh '''
                    git config user.email "pratikkamble122@gmail.com"
                    git config user.name "Pratik Kamble"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" spring-boot-app-manifests/deployment.yml
                    git add spring-boot-app-manifests/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${Github-Token}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:master
                '''
            }
            }
        }


    }
}

