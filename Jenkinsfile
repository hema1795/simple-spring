pipeline {
  environment {
    registryCredential = "17hema"
    PROJECT_ID = 'handy-hexagon-318203'
    CLUSTER_NAME = 'jenkins'
    LOCATION = 'us-central1-c'
    CREDENTIALS_ID = 'handy-hexagon-318203'
    imageName = "springapp"
    registryCredentials = "nexus-artifactory"
    registry = "jokersquotes.com"
    dockerImage = ''
  }
  agent any
  stages {
    
    stage('Checkout Source') {
      steps {
        git url:'https://github.com/hema1795/simple-spring.git', branch:'master'
      }
    }
    
    stage('Code Compile') {
      steps{
        script {
          sh 'mvn clean install'
        }
      }
    }
    stage('Build image') {
      steps{
        script {
           dockerImage = docker.build ("simple-spring:${env.BUILD_ID}")
        }
      }
    }
    
     stage('Push image') {
      steps{
        script {
             docker.withRegistry( 'http://'+registry, registryCredentials )
          {
             dockerImage.push('latest')
             dockerImage.push("${env.BUILD_ID}")
          }
        }
      }
    }
    
    stage('helm build')
    {
      steps{
          git url: 'https://github.com/hema1795/simple-spring.git', branch: 'master'
        script{
          sh '''
          PACKAGE=spring-boot-helm-chart
          helm repo add nexusrepos https://jokersquotes.com/repository/hosted-hosted/ --username admin --password admin
          cd spring-boot-helm-chart
          cat Chart.yaml
          helm package .
          curl -u admin:admin https://jokersquotes.com/repository/hosted-hosted/ --upload-file ${PACKAGE}-*.tgz -v
          '''
        }
      }
    }
        stage('Deploy to GKE test cluster') {
            steps{
              git url: 'https://github.com/hema1795/simple-spring.git', branch: 'master'
              script{
              step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, credentialsId: env.CREDENTIALS_ID])
              sh '''
               PACKAGE=spring-boot-helm-chart
               helm repo add nexusrepos https://jokersquotes.com/repository/hosted-hosted/ --username admin --password admin
               helm repo update
               helm upgrade --install --namespace default/
                --set image.repository=https://jokersquotes.com/repository/docker-registry/simple-spring:${env.BUILD_ID} nexusrepos/${PACKAGE}
                '''
            }
        }
     }
  }
}
