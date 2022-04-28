pipeline {
  agent {
    dockerfile {
      label 'docker'
      filename 'Dockerfile.build'
    }
  }

  options {
    timestamps()
    timeout(time: 2, unit: 'HOURS')
  }

  triggers {
    issueCommentTrigger('.*jenkins test all.*')
  }

  environment {
    NPROC = "0"
  }

  stages {
    stage('Prepare') {
      steps {
        // Cancel any previous job running
        /*
        script {
          def buildNumber = env.BUILD_NUMBER as int
          if (buildNumber > 1) milestone(buildNumber - 1)
          milestone(buildNumber)
        }
        */

        sh '''#!/usr/bin/env bash
        set -exo pipefail
        source ./run-make-check.sh
        FOR_MAKE_CHECK=1 prepare
        '''
      }
    }

    stage('Build') {
      steps {
        sh '''#!/usr/bin/env bash
        set -exo pipefail
        source ./run-make-check.sh
        configure
        build tests
        '''
      }
    }

    stage('Test') {
      parallel {
        stage('Unit Tests') {
          steps {
            sh '''#!/usr/bin/env bash
            set -exo pipefail
            source ./run-make-check.sh
            cd build
            export CHECK_MAKEOPTS="$DEFAULT_MAKEOPTS --no-compress-output -T Test"
            run
            '''
          }
          post {
            always{
              xunit(
                [CTest(
                  deleteOutputFiles: true,
                  failIfNotNew: true,
                  pattern: 'build/Testing/**/Test.xml',
                  skipNoTestFiles: true,
                  stopProcessingIfError: true)
                ]
              )
            }
          }
        }
      } // parallel
    } // stage('Test')

  } // stages
} // pipeline
