pipeline {
    agent any

    environment {
        // Определение переменных окружения для сканирования.
        BURP_START_URL = 'https://eaist.mos.ru'
        IMAGE_WITH_TAG = 'public.ecr.aws/portswigger/dastardly:latest'
        BURP_REPORT_FILE_PATH = 'dastardly-report.xml'
    }

    stages {
        stage("Setup Environment") {
            steps {
                // Очистка рабочего пространства, чтобы избежать конфликтов с предыдущими сборками.
                cleanWs()
                echo "Workspace is now clean."
            }
        }

        stage("Pull Docker Image") {
            steps {
                // Извлечение образа Docker для последующего использования.
                script {
                    docker.image("${env.IMAGE_WITH_TAG}").pull()
                }
                echo "Docker image has been pulled successfully."
            }
        }

        stage("Run Dastardly Scan") {
            steps {
                // Запуск Docker контейнера для выполнения сканирования.
                script {
                    docker.run(
                        "--rm", // Удаляет контейнер после выполнения
                        "--user $(id -u):$(id -g)", // Запуск от текущего UID и GID для соответствия правам доступа
                        "-v ${env.WORKSPACE}:${env.WORKSPACE}:rw", // Примонтировать рабочую директорию
                        "-e BURP_START_URL=${env.BURP_START_URL}", // Передать URL для сканирования
                        "-e BURP_REPORT_FILE_PATH=${env.WORKSPACE}/${env.BURP_REPORT_FILE_PATH}", // Путь для сохранения отчета
                        "${env.IMAGE_WITH_TAG}" // Используемый образ
                    )
                }
                echo "Dastardly scan has been executed."
            }
        }
    }

    post {
        always {
            // Архивация результатов и вывод результатов тестов через JUnit.
            archiveArtifacts artifacts: "${env.BURP_REPORT_FILE_PATH}", onlyIfSuccessful: true
            junit testResults: "${env.WORKSPACE}/${env.BURP_REPORT_FILE_PATH}", allowEmptyResults: true
            echo "Post-build actions completed."
        }
        success {
            echo "Pipeline completed successfully."
        }
        failure {
            echo "Pipeline failed."
            // Отправка уведомления о неудаче на электронную почту.
            mail to: 'your-email@example.com',
                 subject: "Failed Pipeline: ${env.JOB_NAME} Build #${env.BUILD_NUMBER}",
                 body: "There was a failure in the pipeline: ${env.BUILD_URL}"
        }
    }
}
