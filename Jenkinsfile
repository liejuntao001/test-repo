#!/usr/bin/env groovy

tokens = "${JOB_NAME}".tokenize('/')
repoOwner = tokens[tokens.size()-3]
repoName = tokens[tokens.size()-2]

if (BRANCH_NAME == 'master') {
  stage('Check') {
    milestone 0
    gradle null, 'dev', 'clean check sonarqube', false
  }

  milestone 1
  stage('Publish') {
    timeout(time: 6, unit: 'HOURS')
    input message: 'Publish as:', parameters: [[$class: 'ChoiceParameterDefinition', choices: ['milestone', 'rc'] as String[], name: 'publishStage']]
    milestone 2
    gradle null, publishStage, 'clean bintrayUpload tagVersion', false
  }

  if (publishStage == 'rc') {
    stage('Release') {
      input message: 'Release?'
      milestone 3
      gradle null, 'final', 'clean gitPublishPush bintrayUpload tagVersion', false
    }
  }
} else if (CHANGE_ID) {
  stage('Check') {
    milestone 0
    gradle null, 'dev', 'clean check sonarqube', true
  }
} else {
    stage('Unsupported') {
      echo 'Nothing\'s going to happen.'
    }
}

def gradle(scope, stage, args, preview) {
  node {
    dir(repoName) {
      checkout scm
      withCredentials([
        usernamePassword(credentialsId: '29490691-342d-4fa1-b0dc-1e3e27e8e0fa', usernameVariable: 'GRGIT_USER', passwordVariable: 'GRGIT_PASS'),
        usernamePassword(credentialsId: 'fb3c1aa6-6b30-4f48-ba04-9fa0f489bdc5', usernameVariable: 'BINTRAY_USER', passwordVariable: 'BINTRAY_KEY')
      ]) {
        withSonarQubeEnv('SonarQube') {
          try {
            additionalArgs = ''
            if (preview) {
              additionalArgs = "-Dsonar.github.pullRequest=\"${CHANGE_ID}\" -Dsonar.github.repository=\"${repoOwner}/${repoName}\" -Dsonar.github.oauth=\"${GRGIT_PASS}\" -Dsonar.analysis.mode=preview"
            }
            sh "./gradlew --no-daemon -Psemver.stage=${stage} ${args} ${additionalArgs}"
            } finally {
              junit testResults: '**/build/test-results/**/TEST-*.xml', allowEmptyResults: true
            }
          }
        }
    }
  }
}
