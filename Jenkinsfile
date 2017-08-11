@Library('github.com/mozmar/jenkins-pipeline@master')

def loadBranch(String branch) {
  if (fileExists("./Jenkinsfiles/${branch}.yml")) {
    config = readYaml file: "./Jenkinsfiles/${branch}.yml"
    println "config ==> ${config}"
  }
  else {
    config = []
  }

  if (config && config.pipeline && config.pipeline.enabled == false) {
    println "Pipeline disabled."
  }
  else {
    if (config && config.pipeline && config.pipeline.script) {
      println "Loading ./Jenkinsfiles/${config.pipeline.script}.groovy"
      load "./Jenkinsfiles/${config.pipeline.script}.groovy"
    }
    else {
      println "Loading ./Jenkinsfiles/${branch}.groovy"
      load "./Jenkinsfiles/${branch}.groovy"
    }
  }
}

node {
  stage("Prepare") {
    if (env.JOB_NAME.startsWith('kumascript/')) {
      // Running tests for a mdn/kumascript branch
      jenkins_root = 'kumascript'

      // Checkout Kuma project's master branch
      checkout([$class: 'GitSCM',
                branches: [[name: 'refs/heads/master']],
                doGenerateSubmoduleConfigurations: false,
                extensions: [[$class: 'SubmoduleOption',
                              disableSubmodules: false,
                              parentCredentials: false,
                              recursiveSubmodules: true,
                              reference: '',
                              trackingSubmodules: false]],
                submoduleCfg: [],
                userRemoteConfigs: [[url: 'https://github.com/mozilla/kumascript']]
               ])
      dir('kumascript') {
        checkout scm
      }
    }
    else {
      // Running tests for the mozilla/kuma branch
      jenkins_root = '.'
      checkout scm
      sh 'git submodule sync'
      sh 'git submodule update --init --recursive'
    }
    setGitEnvironmentVariables()
    // Set UID to jenkins
    env['UID'] = 1000

    // When checking in a file exists in another directory start with './' or
    // prepare to fail.
    dir(jenkins_root) {
      if (fileExists("./Jenkinsfiles/${env.BRANCH_NAME}.groovy") || fileExists("./Jenkinsfiles/${env.BRANCH_NAME}.yml")) {
        loadBranch(env.BRANCH_NAME)
      }
      else {
        loadBranch("default")
      }
    }
  }
}
