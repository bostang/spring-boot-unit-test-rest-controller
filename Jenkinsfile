pipeline {
  agent any
  tools {
    maven 'Maven 3.8.8'       
    jdk 'Temurin JDK 17'
  }

  environment {
    SONARQUBE_SERVER = 'SonarQube' 
    SONAR_TOKEN = credentials('sonar_token')
    TELEGRAM_BOT_TOKEN = credentials('TG_TOKEN')    // ambil dari Jenkins Credentials
    TELEGRAM_CHAT_ID = credentials('TG_CHAT_ID')
  }
   parameters {
        booleanParam(
            name: 'ENABLE_NOTIFICATIONS',
            defaultValue: true,
            description: 'Enable Telegram notifications'
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
    stage('Send Telegram Notification'){
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