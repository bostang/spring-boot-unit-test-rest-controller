pipeline {
  agent any
  tools {
    maven 'Maven 3.8.8'       
    jdk 'Temurin JDK 17'
  }

  environment {
    SONARQUBE_SERVER = 'SonarQube' 
    SONAR_TOKEN = credentials('sonar_token')
  }

  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/bostang/spring-boot-unit-test-rest-controller', branch: 'master'
      }
    }

    stage('Unit Test & Coverage') {
      steps {
        sh 'mvn package'
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
        }
      }
    }

    stage('Static Code Analysis (SAST) via Sonar') {
      steps {

sh """
            mvn clean compile sonar:sonar \
  -Dsonar.projectKey=e2e-springboot-ci-cd \
  -Dsonar.projectName='e2e-springboot-ci-cd' \
  -Dsonar.host.url=http://sonarqube:9000 \
  -Dsonar.token=${SONAR_TOKEN}
        """
      }
    }
  }

  post {
    success {
      echo "Pipeline berhasil 🚀"
    }
    failure {
      echo "Pipeline gagal 💥"
    }
  }
}