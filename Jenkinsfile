pipeline {
  agent any
  environment {
      DOCKERHUB_CREDENTIALS=credentials('eddy-dockerhub-cred')
      SONARQUBE_CREDENTIALS=credentials('eddy-sonarqube-key')
  }
  stages {
      stage('Build Artifact') {
            steps {
              sh "echo Maven clean job"
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }   

      stage('Unit Tests - JUnit and Jacoco') {
            steps {
              sh "mvn test"
            }
            post {
              always {
                junit 'target/surefire-reports/*.xml'
                jacoco execPattern: 'target/jacoco.exec'
              }
            }
        }

      stage('Sonarqube- SAST') {
        steps {
        sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://eddyattang.westus3.cloudapp.azure.com:9000 -Dsonar.login=sqp_197c589c2f7398b944978be23d472d6a2880c899"
        }
      }
      

      stage('Dockerhub login') {
        steps {
          sh 'echo $DOCKERHUB_CREDENTIALS_PSW |  docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
        }
      }

      stage('Docker push')  {
        steps {
          sh 'printenv'
          sh 'echo Not pushing nada'
          sh 'docker build -t eattang/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push eattang/numeric-app:""$GIT_COMMIT""'
        }
      }

      stage('Kubernetes Deployment - DEV')
      {
        steps {
          withKubeConfig([credentialsId: 'kubeconfig']){
            sh "sed -i 's#replace#eattang/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
            sh "kubectl apply -f k8s_deployment_service.yaml"
          }

        }

      }
    }
}