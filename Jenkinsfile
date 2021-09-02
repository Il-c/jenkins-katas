pipeline {
  agent any
  stages {
    stage('clone down'){
      agent {
        label 'swarm'
      }
      steps {
        stash excludes: '.git', name: 'code'
      }
    }
    stage('Parallel execution') {
      parallel {
        stage('Say Hello') {
          steps {
            sh 'echo "hello world"'
          }
        }

        stage('Build app') {
          options { 
            skipDefaultCheckout(true) 
          }
          agent {
            docker {
              image 'gradle:6-jdk11'
            }
          }
          steps {
            unstash 'code'
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
            sh 'ls -a'
            deleteDir()
            sh 'ls -a'
          }
        }
        stage('test app') {
          options { 
            skipDefaultCheckout(true) 
          }
          agent {
            docker {
              image 'gradle:6-jdk11'
            }
          }
          steps {
            unstash 'code'
            sh 'ci/unit-test-app.sh'
            junit 'app/build/test-results/test/TEST-*.xml'
          }
        }
      }
    }

  }
}