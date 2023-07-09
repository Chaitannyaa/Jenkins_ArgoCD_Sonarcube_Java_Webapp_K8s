pipeline {
  agent {
    docker {
      image 'chaitannyaa/maven-plus-docker'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
    }
  }
  stages {
    stage('Build and Test') {
      steps {
        // build the project and create a JAR file
        sh 'mvn clean package'
      }
    }
    stage('Code Analysis with SonarQube') {
      environment {
        SONAR_URL = "http://54.165.3.3:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "chaitannyaa/java_awesome-cicd:${BUILD_NUMBER}"
        REGISTRY_CREDENTIALS = credentials('dockerHub')
      }
      steps {
        script {
            sh 'docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "dockerHub") {
                dockerImage.push()
            }
        }
      }
    }
    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "Jenkins_ArgoCD_Sonarcube_Java_Webapp_K8s"
            GIT_USER_NAME = "Chaitannyaa"
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "crmg26696@gmail.com"
                    git config user.name "Chaitannyaa Gaikwad"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" manifests/deployment.yml
                    git add -f manifests/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
  }
}
