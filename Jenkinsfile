pipeline {
  agent {
    docker {
      image 'mrmarga/maven-docker-agent:v1'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
    }
    
  }
  stages {

    stage('Checkout') {
      steps {
        sh 'echo passed'
        //git branch: 'main', url: 'https://github.com/MrMarga/cicd-jenkins.git'
      }
    }

    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        // build the project and create a JAR file
        sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && mvn clean package'
      }
    }

    stage('Static Code Analysis'){// {
      //environment {
      //  SONAR_URL = "http://192.168.100.58:9000"
    //  }
      steps {
        sh "echo By Passsing"
     //   withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
       //   sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
     }
      
    stage('Build and Push Docker Image') {
       environment{DOCKERHUB_CREDENTIALS = credentials('docker-cred'),}  
      steps {
            sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && docker build . -t mrmarga/ultimate-cicd && docker tag mrmarga/ultimate-cicd mrmarga/ultimate-cicd:${BUILD_NUMBER}
            && echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin 
            && docker push $DOCKERHUB_CREDENTIALS_USR/ultimate-cicd:${BUILD_NUMBER}'
             } 
    }

    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "Ultimate-CiCd"
            GIT_USER_NAME = "mrmarga"
        }

        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "margaghale31@gmail.com"
                    git config user.name "mrmarga"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/deployment.yml
                    git add java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }  
    }  
 }

}
