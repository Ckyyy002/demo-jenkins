pipeline {
    agent any

    environment {
        SONAR_TOKEN     = credentials('sonarqube-token')
        DISCORD_WEBHOOK = credentials('discord-webhook-url')
        SONAR_PROJECT_KEY = 'demo-jenkins'
        PATH = "/usr/local/go/bin:${env.PATH}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo '📥 Mengambil kode dari GitHub...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo '🔨 Menjalankan Build...'
                sh 'go version'
                sh 'go build ./...'
            }
        }

        stage('Test') {
            steps {
                echo '🧪 Menjalankan Unit Test...'
                sh 'go test ./... -v'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo '🔍 Menjalankan SonarQube Analysis...'
                withSonarQubeEnv('SonarQube') {
                    sh """
                        sonar-scanner \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.projectName='Demo Jenkins' \
                          -Dsonar.sources=. \
                          -Dsonar.exclusions=**/*_test.go \
                          -Dsonar.host.url=http://57.158.98.73:9000 \
                          -Dsonar.token=${SONAR_TOKEN}
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo '🚦 Menunggu Quality Gate SonarQube...'
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                echo '🚀 Deploying...'
                sh 'go build -o app ./...'
                echo '✅ Deploy berhasil!'
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline BERHASIL!'
            script {
                def msg = """{"embeds": [{"title": "✅ Build Berhasil","description": "**Job:** ${env.JOB_NAME}\\n**Build:** #${env.BUILD_NUMBER}\\n**Branch:** ${env.GIT_BRANCH ?: 'N/A'}\\n**URL:** ${env.BUILD_URL}","color": 3066993}]}"""
                sh "curl -s -X POST '${DISCORD_WEBHOOK}' -H 'Content-Type: application/json' -d '${msg}'"
            }
        }
        failure {
            echo '❌ Pipeline GAGAL!'
            script {
                def msg = """{"embeds": [{"title": "❌ Build Gagal","description": "**Job:** ${env.JOB_NAME}\\n**Build:** #${env.BUILD_NUMBER}\\n**Branch:** ${env.GIT_BRANCH ?: 'N/A'}\\n**URL:** ${env.BUILD_URL}","color": 15158332}]}"""
                sh "curl -s -X POST '${DISCORD_WEBHOOK}' -H 'Content-Type: application/json' -d '${msg}'"
            }
        }
        always {
            echo '🏁 Pipeline selesai dieksekusi.'
            cleanWs()
        }
    }
}
