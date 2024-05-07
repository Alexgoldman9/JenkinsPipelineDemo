pipeline {
    agent any

    environment {
        BURP_START_URL = 'https://eaist.mos.ru'
        IMAGE_WITH_TAG = 'public.ecr.aws/portswigger/dastardly:latest'
        BURP_REPORT_FILE_PATH = 'dastardly-report.xml'
    }

    stages {
        stage("Setup Environment") {
            steps {
                cleanWs()
            }
        }

        stage("Pull Docker Image") {
            steps {
                script {
                    docker.image("${env.IMAGE_WITH_TAG}").pull()
                }
            }
        }

        stage("Run Dastardly Scan") {
            steps {
                script {
                    docker.run(
                        "--rm",
                        "--user $(id -u):$(id -g)",
                        "-v ${env.WORKSPACE}:${env.WORKSPACE}:rw",
                        "-e BURP_START_URL=${env.BURP_START_URL}",
                        "-e BURP_REPORT_FILE_PATH=${env.WORKSPACE}/${env.BURP_REPORT_FILE_PATH}",
                        "${env.IMAGE_WITH_TAG}"
                    )
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: "${env.BURP_REPORT_FILE_PATH}", onlyIfSuccessful: true
            junit testResults: "${env.WORKSPACE}/${env.BURP_REPORT_FILE_PATH}", allowEmptyResults: true
        }
        success {
        }
        failure {
            mail to: 'your-email@example.com',
                 subject: "Failed Pipeline: ${env.JOB_NAME} Build #${env.BUILD_NUMBER}",
                 body: "There was a failure in the pipeline: ${env.BUILD_URL}"
        }
    }
}
