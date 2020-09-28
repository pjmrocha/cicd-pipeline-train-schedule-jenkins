pipeline {
  agent any
  environment {
    DOCKER_IMAGE_NAME = "pjmrocha/train-schedule"
  }
  stages {
    stage ('Build') {
      steps {
        echo 'Running build automation'
        sh './gradlew build --no-daemon'
        archiveArtifacts artifacts: 'dist/trainSchedule.zip'
      }
    }
    stage('Build Docker Image') {
      when {
        branch 'train-deploy-k8-canary'
      }
      steps {
        script {
          app = docker.build(DOCKER_IMAGE_NAME)
          app.inside {
            sh 'echo Hello, World!'
          }
        }
      }
    }
    stage('Push Docker Image') {
      when {
        branch 'train-deploy-k8-canary'
      }
      steps {
        script {
          docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
          }
        }
      }
    }
    stage('CanaryDeploy') {
      when {
        branch 'train-deploy-k8-canary'
      }
      environment { 
        CANARY_REPLICAS = 1
      }
      steps {
        kubernetesDeploy(
          kubeconfigId: 'kubeconfig',
          configs: 'train-schedule-kube-canary.yml',
          enableConfigSubstitution: true
        )
      }
    }
  }
}
