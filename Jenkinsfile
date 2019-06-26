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
        sh "wget -O /home/jenkins/gitleaks.sh -q https://raw.githubusercontent.com/trilogy-group/gitleaks-ci/master/gitleaks.sh && chmod +x /home/jenkins/gitleaks.sh && /home/jenkins/gitleaks.sh 1"
        sh "mvn versions:set -DnewVersion=$PREVIEW_VERSION"
        sh "mvn install"
      }
    }
    stage('Build Release') {
      when {
        branch 'master'
      }
      steps {
        git 'https://github.com/denispalnitskytest/test-jx-sequence.git'

        // so we can retrieve the version in later steps
        sh "echo \$(jx-release-version) > VERSION"
        sh "mvn versions:set -DnewVersion=\$(cat VERSION)"
        sh "jx step tag --version \$(cat VERSION)"
        sh "git fetch --depth=2 -q && wget -O /home/jenkins/gitleaks.sh -q https://raw.githubusercontent.com/trilogy-group/gitleaks-ci/master/gitleaks.sh && chmod +x /home/jenkins/gitleaks.sh && /home/jenkins/gitleaks.sh 2"
        sh "echo $AWS_REGION"
        sh "mvn clean install -DskipTests=true"
        sh "mvn surefire-report:report-only"
        sh "jx step stash --classifier tests --pattern "target/site/surefire-report.html" --verbose"
        sh "mvn cobertura:cobertura -Dcobertura.report.format=xml"
        sh "mvn com.googlecode.maven-download-plugin:download-maven-plugin:1.4.2:artifact -DgroupId=org.eluder.coveralls -DartifactId=coveralls-maven-plugin -Dversion=4.3.0"
        sh "mvn cobertura:cobertura org.eluder.coveralls:coveralls-maven-plugin:report -DrepoToken=$COVERALLS_TOKEN"
        dir('react-app')
        sh "npm i @babel/core @babel/plugin-proposal-class-properties @babel/preset-env @babel/preset-react babel-loader istanbul-instrumenter-loader"
        sh "karma start"
        sh "jx step stash --classifier tests --pattern "tests-tesults/units.html" --verbose"
      }
    }
  }
}
