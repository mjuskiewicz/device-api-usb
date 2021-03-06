Boolean ISMASTER = BRANCH_NAME == "master" ? true : false
String cronTrigger = ISMASTER ? 'H H(0-2) * * *' : ''
Map stage_node_info = [:]

pipeline {
  agent none
  options {
    skipDefaultCheckout()
    disableConcurrentBuilds()
    buildDiscarder(logRotator(numToKeepStr: '50', artifactNumToKeepStr: '50'))
  }
  triggers { cron(cronTrigger) }
  parameters {
    booleanParam(name: 'ReleaseBuild', defaultValue: false, description: 'Should release the built platform')
  }
  environment {
    GIT_BRANCH = "${BRANCH_NAME}"
    NODE_VERSION="12.4.0"
    RELEASE="${params.ReleaseBuild}"
  }
  stages {
    stage ('Tooling') {
      failFast true
      agent { label 'master' }
      steps {
          echo "Running on $NODE_NAME"
          timeout(30) {
            cleanWs()
            copyArtifacts filter: '**/tools.tar.gz', flatten: true, projectName: 'boxfish-multibranch-pipeline/master', selector: lastSuccessful(), target: '.'
            sh 'tar xzf tools.tar.gz'
            stash includes: "tools/**/*", name: 'tools'
          }
      }
    }
    stage('Build') {
      failFast true
      agent { label "docker && qemu" }
      steps {
        echo "Running on $NODE_NAME"
        timeout(30) {
          ws("/var/jenkins/workspace/executor$EXECUTOR_NUMBER") {
            cleanWs()
            checkout scm

            unstash 'tools'

            withCredentials([string(credentialsId: 'azuredeviceapiusbbinariesStorageToken', variable: 'AZURE_STORAGE_ACCESS_KEY')]) {
              withEnv([ 'AZURE_STORAGE_ACCOUNT=deviceapiusb' ]) {
                ansiColor('xterm') {
                  sh 'scripts/build_on_qemu.sh ~/win10'
                }
              }
            }
          }
        }
      }
      post {
        always {
          cleanWs()
          script { stage_node_info["$STAGE_NAME"] = "$NODE_NAME"}
        }
      }
    }
  }
}
