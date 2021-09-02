/* groovylint-disable NestedBlockDepth */
pipeline {
  agent any

  environment {
    docker_username = 'lehtinei'
  //DOCKERCREDS = credentials('Docker_credentials')
  }

  stages {
    stage('clone down') {
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
            stash excludes: '.git', name: 'code'
            archiveArtifacts 'app/build/libs/'
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
    stage ('push docker app') {
      environment {
        DOCKERCREDS = credentials('Dockerhub_credentials') //use the credentials just created in this stage
      }
      when {
        beforeAgent true
        branch 'master'
      }
      steps {
        input message: 'Do you want to push to docker hub?', ok: 'Yes'
        unstash 'code' //unstash the repository code
        sh 'ci/build-docker.sh'
        sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
        sh 'ci/push-docker.sh'
      }
    }
    stage('component test') {
          options {
            skipDefaultCheckout(true)
          }
          agent {
            docker {
              image 'gradle:6-jdk11'
            }
          }
          when { 
            beforeAgent true
            branch pattern: "^(?!dev/).+", comparator: "REGEXP"}
          steps {
            unstash 'code'
            sh 'ci/component-test.sh'
            //junit 'app/build/test-results/test/TEST-*.xml'
          }
        }
  }
  post {
    cleanup {
        deleteDir() /* clean up our workspace */
    }
  }
}
