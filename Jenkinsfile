pipeline {
  agent any
  environment {
    ORG = 'denispalnitskytest'
    APP_NAME = 'test-jx-sequence'
    CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
    DOCKER_REGISTRY_ORG = 'denispalnitskytest'
  }
  stages {
    stage('CI Build and push snapshot') {
      when {
        branch 'PR-*'
      }
      environment {
        PREVIEW_VERSION = "0.0.0-SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER"
        PREVIEW_NAMESPACE = "$APP_NAME-$BRANCH_NAME".toLowerCase()
        HELM_RELEASE = "$PREVIEW_NAMESPACE".toLowerCase()
      }
      steps {
        sh "mvn versions:set -DnewVersion=$PREVIEW_VERSION"
        sh "mvn install"
      }
    }
    stage('Build Release') {
      when {
        branch 'master'
      }
      steps {

        // so we can retrieve the version in later steps
        sh "echo \$(jx-release-version) > VERSION"
        sh "mvn versions:set -DnewVersion=\$(cat VERSION)"
        sh "jx step tag --version \$(cat VERSION)"
        sh "echo Python1 > step-1"
        sh "ls -a"
        container('gcr.io/jenkinsxio/builder-nodejs')
        sh "ls -a"
        sh "echo Python2 > step-3"
        sh "ls -a"
      }
    }
  }
}
