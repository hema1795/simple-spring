pipeline {
  environment {
    //registryCredential = "17hema"
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
            dockerImage = docker.build("handy-hexagon-318203/snapshot")
      }
    }
    } 
     stage('Push image') {
      steps{
        script {
             docker.withRegistry( 'https://gcr.io', 'gcr:handy-hexagon-318203' )
          {
             dockerImage.push("${env.BUILD_NUMBER}")
             dockerImage.push('latest')
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
          PACKAGE=sample-chart
          helm repo add nexusrepos https://jokersquotes.com/repository/hosted-hosted/ --username admin --password admin
          cd sample-chart
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
              sh '''
               export project=handy-hexagon-318203
               export key=sample.json
               export zone=us-central1-c
               export cluster=jenkins
               gcloud auth activate-service-account --project=${project} --key-file=${key}
               gcloud container clusters get-credentials ${cluster} --zone ${zone} --project ${project}
               PACKAGE=sample-chart
               helm repo add nexusrepos  https://jokersquotes.com/repository/hosted-hosted/ --username admin --password admin
               cd sample-chart
               helm repo update
               PACKAGE=nexusrepos/sample-chart
               helm repo update
               helm upgrade --install sample-chart -f values.yaml ${PACKAGE} -n default
                '''
            }
        }
     }
  }
}
