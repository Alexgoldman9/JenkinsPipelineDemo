pipeline {
    agent any

    environment {
        // Используем рекомендованные имена переменных
        BURP_START_URL = 'https://eaist.mos.ru'
        IMAGE_WITH_TAG = 'public.ecr.aws/portswigger/dastardly:latest'
        BURP_REPORT_FILE_PATH = 'dastardly-report.xml'
    }

    stages {
        stage("Setup Environment") {
            steps {
                cleanWs() // Очищаем рабочую область перед каждым запуском
            }
        }

        stage("Pull Docker Image") {
            steps {
                sh "docker pull ${IMAGE_WITH_TAG}" // Загружаем последнюю версию образа Docker
            }
        }

        stage("Run Dastardly Scan") {
            steps {
                script {
                    // Запускаем Docker контейнер для сканирования
                    def user = sh(script: 'id -u', returnStdout: true).trim()
                    def command = """
                    docker run --rm --user ${user} -v ${WORKSPACE}:${WORKSPACE}:rw \\
                    -e BURP_START_URL=${BURP_START_URL} \\
                    -e BURP_REPORT_FILE_PATH=${WORKSPACE}/${BURP_REPORT_FILE_PATH} \\
                    ${IMAGE_WITH_TAG}
                    """
                    sh command
                }
            }
        }
    }

    post {
        always {
            // Публикуем результаты и архивируем отчет
            junit testResults: "${WORKSPACE}/${BURP_REPORT_FILE_PATH}", allowEmptyResults: true
            archiveArtifacts artifacts: "${BURP_REPORT_FILE_PATH}"
        }
    }
}
