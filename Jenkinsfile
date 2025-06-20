def version = null

pipeline {
  agent any
  triggers {
    githubPush()
    pollSCM('H H * * *')
  }
  environment {
    MAIL_RECIPIENTS = 'dev+tests-reports@wazo.community'
  }
  options {
    skipStagesAfterUnstable()
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '10'))
    throttleJobProperty(
      categories: ['asterisk-debug'],
      throttleEnabled: true,
      throttleOption: 'category'
    )
  }
  stages {
    stage ('Prepare') {
      steps {
        script {
          version = sh(script: 'dpkg-parsechangelog --show-field version', returnStdout: true).trim()
          currentBuild.displayName = "${JOB_NAME} ${version}"
          currentBuild.description = "Build Debian package ${JOB_NAME} ${version}"
        }
      }
    }
    stage('Debian build and deploy') {
      steps {
        build job: 'build-package-multi-arch', parameters: [
          string(name: 'PACKAGE', value: "${JOB_NAME}"),
          string(name: 'VERSION', value: "${version}"),
        ]
      }
    }
  }
  post {
    failure {
      emailext to: "${MAIL_RECIPIENTS}", subject: '${DEFAULT_SUBJECT}', body: '${DEFAULT_CONTENT}'
    }
    fixed {
      emailext to: "${MAIL_RECIPIENTS}", subject: '${DEFAULT_SUBJECT}', body: '${DEFAULT_CONTENT}'
    }
  }
}
