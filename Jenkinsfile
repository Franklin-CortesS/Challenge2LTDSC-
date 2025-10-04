pipeline {
  agent any

  environment {
    IMAGE_NAME = "demo-micro"
    DOCKERHUB_NAMESPACE = "franklincs24"         
    REGISTRY = "docker.io"
    JAVA_HOME = tool name: 'JDK17', type: 'hudson.model.JDK'
    MAVEN_HOME = tool name: 'M3', type: 'hudson.tasks.Maven$MavenInstallation'
    PATH = "${JAVA_HOME}/bin:${MAVEN_HOME}/bin:${env.PATH}"
  }

  options {
    timestamps()
    ansiColor('xterm')
    disableConcurrentBuilds()
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build JAR') {
      steps {
        bat 'mvn -B -DskipTests clean package'
      }
      post {
        success {
          archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
      }
    }

    stage('Build & Push Image') {
      steps {
        script {
          def tag = env.BUILD_NUMBER
          def image = docker.build("${DOCKERHUB_NAMESPACE}/${IMAGE_NAME}:${tag}")
          docker.withRegistry("https://${REGISTRY}", 'dockerhub-creds') {
            image.push()
            image.push('latest')
          }
        }
      }
    }
  }

  post {
    success {
      echo "Imagen publicada: ${env.REGISTRY}/${DOCKERHUB_NAMESPACE}/${IMAGE_NAME}:${env.BUILD_NUMBER}"
    }
    failure {
      echo "Build fallido. Revisar logs."
    }
  }
}
