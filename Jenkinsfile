#!/usr/bin/env groovy

def deployableBranches = ["master", "staging"]

podTemplate(
  label: "pqmt-test",
  inheritFrom: "default",
  containers: [
    containerTemplate(name: 'gcloud',
                      image: 'google/cloud-sdk:latest',
                      ttyEnabled: true,
                      command: 'cat')],
  volumes: [
    hostPathVolume(hostPath: '/var/run/docker.sock',
                   mountPath: '/var/run/docker.sock'),
    hostPathVolume(hostPath: '/usr/bin/docker',
                   mountPath: '/usr/bin/docker')])
{
  node("pqmt-test") {
    stage("Checkout source") {
      checkout scm
    }


    containerTag = "us.gcr.io/oak-backend/pqmt-air-docker/${env.BRANCH_NAME}:${env.BUILD_NUMBER}"

    stage("Build") {
      container("gcloud") {
        sh("gcloud docker -- build -t ${containerTag} .")
      }
    }

    container("gcloud") {
      if (deployableBranches.contains(env.BRANCH_NAME)) {
        stage ("Publish Docker containers") {
          sh("gcloud docker -- push ${containerTag}")

          sh("mkdir -p ./output")
          sh("chmod a+rwx ./output")
          writeFile(file: "output/container-tag.txt", text: containerTag)
          archiveArtifacts(artifacts: "output/*.txt")
        }
      }
      stage("Cleanup")  {
        sh("gcloud docker -- rmi ${containerTag}")
      }
    }
  }
}

