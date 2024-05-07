pipeline {
    agent any // Определяет, что пайплайн может выполняться на любом доступном агенте.

    environment {
        // Определение переменных окружения для конкретного сайта.
        BURP_START_URL = 'https://eaist.mos.ru'
        IMAGE_WITH_TAG = 'public.ecr.aws/portswigger/dastardly:latest'
        JUNIT_TEST_RESULTS_FILE = 'dastardly-report.xml'
    }

    stages {
        stage("Prepare Environment") {
            steps {
                // Очистка рабочей области перед началом работы.
                cleanWs()
                echo "Workspace cleaned"
            }
        }

        stage("Docker Pull Dastardly from Burp Suite container image") {
            steps {
                // Подтягивание последней версии Docker образа.
                script {
                    docker.image("${IMAGE_WITH_TAG}").pull()
                }
                echo "Docker image pulled"
            }
        }

        stage("Docker run Dastardly from Burp Suite Scan") {
            steps {
                // Запуск сканирования с использованием Docker.
                script {
                    docker.run(
                        "--rm",
                        "--user $$(id -u)", // Двойной доллар для корректного экранирования в Groovy
                        "-v ${WORKSPACE}:${WORKSPACE}:rw",
                        "-e BURP_START_URL=${BURP_START_URL}",
                        "-e BURP_REPORT_FILE_PATH=${WORKSPACE}/${JUNIT_TEST_RESULTS_FILE}",
                        "${IMAGE_WITH_TAG}"
                    )
                }
                echo "Dastardly scan completed"
            }
        }
    }

    post {
        always {
            // Архивация результатов теста и обработка результатов с помощью JUnit.
            archiveArtifacts artifacts: "${JUNIT_TEST_RESULTS_FILE}", onlyIfSuccessful: true
            junit testResults: "${WORKSPACE}/${JUNIT_TEST_RESULTS_FILE}", allowEmptyResults: true
            echo "Post actions completed"
        }
        success {
            echo "Pipeline completed successfully."
        }
        failure {
            echo "Pipeline failed."
            mail to: 'you@example.com', subject: "Failed Pipeline: ${env.JOB_NAME} [${env.BUILD_NUMBER}]", body: "Something went wrong in ${env.BUILD_URL}"
        }
    }
}
