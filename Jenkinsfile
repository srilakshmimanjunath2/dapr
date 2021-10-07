@Library('jenkins.shared.library') _

pipeline {
  agent {
    label 'ubuntu_docker_label'
  }
  tools {
    go "Go 1.16"
  }
    options {
        checkoutToSubdirectory('src/github.com/infobloxopen/dapr')
  }
  environment {
    GOPATH = "$WORKSPACE"
    DIRECTORY = "src/github.com/infobloxopen/dapr"
    DOCKER_IMAGE = "infoblox/dapr"
    
  }
  stages {
    stage("Setup") {
      steps {
        prepareBuild()
      }
    }
   stage("Test") {
      steps {
        sh "cd $DIRECTORY && make test"
      }
    }
   stage("build-and-archive-binaries-darwin-amd64") {
      steps {
        sh "cd $DIRECTORY && make tidy && make release GOOS='darwin' GOARCH='amd64' "
      }
     	  
	  
    }
   stage("build-and-archive-binaries-linux-arm64"){
        steps {
          sh "cd $DIRECTORY && make tidy && make release GOOS='linux' GOARCH='arm64' "
        }

      }
    stage("build-and-archive-binaries-linux-amd64"){
         steps {
          sh "cd $DIRECTORY && make tidy && make release GOOS='linux' GOARCH='amd64' "
        }
      }
    stage("Build-And-Push-Docker") {
       steps {
        withDockerRegistry([credentialsId: "dockerhub-bloxcicd", url: ""]) {
          sh "cd $DIRECTORY && make docker-push GOOS='darwin' GOARCH='amd64' && make docker-push GOOS='linux' GOARCH='arm64' && make docker-push GOOS='linux' GOARCH='amd64' "
          
        }
      }
    }
    stage("Push merge") {
       when {
          not { changeRequest() }
          not { buildingTag() }
       }
       steps {
          withDockerRegistry([credentialsId: "dockerhub-bloxcicd", url: ""]) {
             sh 'cd $DIRECTORY && make docker-push USERNAME="Jenkins"'
             sh 'cd $DIRECTORY && make docker-push'
             sh 'cd $DIRECTORY && make docker-manifest-create'
             sh 'cd $DIRECTORY && make docker-publish'
             sh 'cd $DIRECTORY && make check-windows-version'
             sh 'cd $DIRECTORY && make docker-windows-base-build'
             sh 'cd $DIRECTORY && make docker-windows-base-push'
             
          }
          // AWS_IAM_CI_CD_INFRA
          withAWS(credentials: "CICD_HELM", region: "us-east-1") {
            sh "cd $DIRECTORY && make push-chart"
          }
          dir("${WORKSPACE}/${DIRECTORY}") {
            archiveArtifacts artifacts: 'repo/*.tgz'
            archiveArtifacts artifacts: 'build/build.properties'
          }
       }
    }
    stage("Push Release/Tag") {
       when {
          buildingTag()
       }
       steps {
          withDockerRegistry([credentialsId: "dockerhub-bloxcicd", url: ""]) {
             sh 'cd $DIRECTORY && make push IMAGE_VERSION=${TAG_NAME}'
          }
          // AWS_IAM_CI_CD_INFRA
          withAWS(credentials: "CICD_HELM", region: "us-east-1") {
            sh "cd $DIRECTORY && make push-chart"
          }
          dir("${WORKSPACE}/${DIRECTORY}") {
            archiveArtifacts artifacts: 'repo/*.tgz'
            archiveArtifacts artifacts: 'build/build.properties'
          }
       }
    }
  }
  post {
    success {
      finalizeBuild()
    }
    cleanup {
      sh "cd $DIRECTORY && make clean"
    }
  }
}
