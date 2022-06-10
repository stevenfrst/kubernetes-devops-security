pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }   
      stage('unit test') {
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
    stage('Mutation Tests - PIT') {
      steps {
        sh "mvn org.pitest:pitest-maven:mutationCoverage"
      }
      post {
        always {
          pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
        }
      }
    }
    
    stage('sonartest') {
      steps {
        sh "mvn clean verify sonar:sonar \
    -Dsonar.projectKey=numeric-application \
    -Dsonar.host.url=http://localhost:9000 \
    -Dsonar.login=7f30385741e9f8a8331ad9842048ce8b67024f93"
        }
    }   
     stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "555279fb-2b24-4675-b6dc-96ed87862158", url: ""]) {
          sh 'printenv'
          sh 'docker build -t stevenfrst/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push stevenfrst/numeric-app:""$GIT_COMMIT""'
        }
      }
    }
      stage('Kubernetes Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 's#replace#siddharth67/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    }
  }
}
