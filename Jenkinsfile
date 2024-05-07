pipeline {
  agent any
  environment {
    BURP_START_URL='https://eaist.mos.ru'
    IMAGE_WITH_TAG='public.ecr.aws/portswigger/dastardly:latest'
    JUNIT_TEST_RESULTS_FILE='dastardly-report.xml'
  }
  stages {
    stage ("Docker Pull Dastardly from Burp Suite container image") {
      steps {
        sh "docker pull ${IMAGE_WITH_TAG}"
      }
    }
    stage ("Docker run Dastardly from Burp Suite Scan") {
      steps {
        cleanWs()
        sh """
          docker run --rm --user $(id -u) -v ${WORKSPACE}:${WORKSPACE}:rw \
          -e BURP_START_URL=${BURP_START_URL} \
          -e BURP_REPORT_FILE_PATH=${WORKSPACE}/${JUNIT_TEST_RESULTS_FILE} \
          ${IMAGE_WITH_TAG}
        """
      }
    }
  }
  post {
    always {
      junit testResults: "${WORKSPACE}/${JUNIT_TEST_RESULTS_FILE}", skipPublishingChecks: true
    }
  }
}
