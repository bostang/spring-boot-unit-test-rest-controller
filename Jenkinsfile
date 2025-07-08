pipeline {
  agent any
  tools {
    maven 'Maven 3.8.8'       
    jdk 'Temurin JDK 17'
  }

  environment {
    // sonarQube
    SONARQUBE_SERVER = 'SonarQube' 
    SONAR_TOKEN = credentials('sonar_token')

    // Telegram
    TELEGRAM_BOT_TOKEN = credentials('TG_TOKEN')    // ambil dari Jenkins Credentials
    TELEGRAM_CHAT_ID = credentials('TG_CHAT_ID')

    // Andrew:
      // TELEGRAM_BOT_TOKEN = '8029797501:AAHvAp4KV1KUabDAFN-Kalc58MDKm1sgQyc'
      // TELEGRAM_CHAT_ID = '2052628431'

    // Sri:
      // TELEGRAM_TOKEN = '7631091755:AAENejqAcoa_EIhR5lPO_M9HiCwaVOWTw0A'
      // TELEGRAM_CHAT_ID = '1252569426'

    // Dockerhub
    IMAGE_NAME = 'bostang/springboot-e2e-ci-cd'
    IMAGE_TAG = 'latest'
  }
   parameters {
        booleanParam(
            name: 'ENABLE_NOTIFICATIONS',
            defaultValue: true,
            description: 'Enable Telegram notifications'
        )
        booleanParam(
            name: 'ENABLE_TEST',
            defaultValue: true,
            description: 'Enable Unit Test & SAST'
        )
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
      // -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
          // sebagai Konfigurasi Path Coverage ke Sonar
      // mvn verify dipilih alih-alih mvn compile, agar testing dan laporan coverage dijalankan sebelum sonar:sonar.
        if (params.ENABLE_TEST) {
      steps {
sh """
            mvn clean verify sonar:sonar \
  -Dsonar.projectKey=e2e-springboot-ci-cd \
  -Dsonar.projectName='e2e-springboot-ci-cd' \
  -Dsonar.host.url=http://sonarqube:9000 \
  -Dsonar.token=${SONAR_TOKEN} \
  -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
        """
// sh """
//             mvn clean compile sonar:sonar \
//   -Dsonar.projectKey=e2e-springboot-ci-cd \
//   -Dsonar.projectName='e2e-springboot-ci-cd' \
//   -Dsonar.host.url=http://sonarqube:9000 \
//   -Dsonar.token=${SONAR_TOKEN} \
//         """
      }
        }
    }
    stage('Build Docker Image') {
  steps {
    script {
      sh """
        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
      """
    }
  }
}

    stage('Send Telegram Notification'){
                  steps {
                script {
      if (params.ENABLE_NOTIFICATIONS) {
        def startMessage = """
🚀 <b>CI/CD Pipeline Started</b>
━━━━━━━━━━━━━━━━━━━━━━━
📋 <b>Job:</b> ${env.JOB_NAME}
🔢 <b>Build:</b> #${env.BUILD_NUMBER}
🌿 <b>Branch:</b> ${env.BRANCH_NAME ?: 'main'}
🏷️ <b>Tag:</b> ${env.FINAL_TAG}
👤 <b>Triggered by:</b> ${env.BUILD_USER ?: 'System'}
⏰ <b>Started at:</b> ${new Date().format('yyyy-MM-dd HH:mm:ss')}
🔗 <b>Console:</b> ${env.BUILD_URL}console
                    """
          sendTelegramMessage(startMessage)
      }
    }
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

def sendTelegramMessage(String message) {
    if (!params.ENABLE_NOTIFICATIONS) {
        return
    }
    
    try {
        echo "📱 Sending Telegram notification:"
        echo "Message: ${message}"
        echo "✅ Telegram notification sent successfully (simulated)"
    } catch (Exception e) {
        echo "❌ Telegram notification error: ${e.getMessage()}"
    }

    try {
        def encodedMessage = message.replaceAll('"', '\\\\"')
        def maxRetries = 3
        def retryCount = 0
        def success = false
    
        while (retryCount < maxRetries && !success) {
            try {
                def response = sh(
                    script: """
                        curl -s -X POST https://api.telegram.org/bot\${TELEGRAM_BOT_TOKEN}/sendMessage \
                            -d chat_id=\${TELEGRAM_CHAT_ID} \
                            -d text="${encodedMessage}" \
                            -d parse_mode=HTML \
                            -w "HTTP_CODE:%{http_code}"
                    """,
                    returnStdout: true
                ).trim()
                
                if (response.contains('HTTP_CODE:200')) {
                    echo "✅ Telegram notification sent successfully"
                    success = true
                } else {
                    throw new Exception("HTTP error: ${response}")
                }
            } catch (Exception e) {
                retryCount++
                echo "⚠️ Telegram notification attempt ${retryCount} failed: ${e.getMessage()}"
                if (retryCount < maxRetries) {
                    sleep(time: 5, unit: 'SECONDS')
                }
            }
        }
        
        if (!success) {
            echo "❌ Failed to send Telegram notification after ${maxRetries} attempts"
        }
    } catch (Exception e) {
        echo "❌ Telegram notification error: ${e.getMessage()}"
    }
}