pipeline {
  agent {
    node {
      label 'nodejs'
    }

  }
  stages {
    stage('preamble') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject() {
              echo "Using project: ${openshift.project()}"
            }
          }
        }

      }
    }

    stage('CheckOut') {
      steps {
        sh '''#!/bin/bash
            oc apply -f Yamls/
            '''
      }
    }

    stage('build') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject() {
              def builds = openshift.selector("bc", "drupal").startBuild()
              timeout(5) {
                builds.untilEach(1) {
                  return (it.object().status.phase == "Complete")
                }
              }
            }
          }
        }

      }
    }

    stage('deploy') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject() {
              // openshift.selector("dc", "${templateName}").rollout().latest()
              def rm = openshift.selector("dc", "drupal").rollout().latest()
              timeout(5) {
                openshift.selector("dc", templateName).related('pods').untilEach(1) {
                  return (it.object().status.phase == "Running")
                }
              }
            }
          }
        }

      }
    }

  }
  options {
    timeout(time: 20, unit: 'MINUTES')
  }
}