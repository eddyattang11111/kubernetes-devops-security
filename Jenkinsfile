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
        }
      // stage('Vulnerability scan - Docker') {
      //   steps {
      //     sh "mvn dependency-check:check"
      //   }
      // }

      // stage('Vulnerability scan - Docker') {
      //   steps {
      //     parallel {

      //       "Dependency Scan": {
      //           echo "Dependency scan"
      //           // sh "mvn dependency-check:check"
      //       },
      //       "Trivy scan": {
      //           echo  "Trivy scan"
      //         //  sh "bash trivy-docker-image-scan.sh"
      //       }
      //     }
          
      //   }
      // }

      stage('run-parallel-branches') {
        steps {
          parallel(
            a: {
              echo "This is branch a"
            },
            b: {
              echo "This is branch b"
            }
          )
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
    post {
      always {
            junit 'target/surefire-reports/*.xml'
            jacoco execPattern: 'target/jacoco.exec'
            dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
      }
      // success {

      // }

      // failure {

      // }
    }
}