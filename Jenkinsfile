pipeline {
  agent any
  stages {

    stage('Build') {
      agent {
        label "lead-toolchain-skaffold"
      }
      steps {
        container('skaffold') {
          sh "skaffold build --file-output=image.json"
          stash includes: 'image.json', name: 'build'
          sh "rm image.json"
        }
      }
    }

    stage("Deploy to Staging") {
      agent {
        label "lead-toolchain-skaffold"
      }
      when {
          branch 'master'
      }
      environment {
        TILLER_NAMESPACE = "${env.stagingNamespace}"
        ISTIO_DOMAIN = "${env.stagingDomain}"
        PRODUCT_NAME = "${env.product}"
      }
      steps {
        container('skaffold') {
          unstash 'build'
          sh "skaffold deploy -a image.json -n ${TILLER_NAMESPACE}"
          stageMessage "Successfully deployed to staging:\nspringtrader-${env.product}.${env.stagingDomain}/spring-nanotrader-web/"
        }
      }
    }


    stage ('Manual Ready Check') {
      agent none
      when {
        branch 'master'
      }
      options {
        timeout(time: 30, unit: 'MINUTES')
      }
      input {
        message 'Deploy to Production?'
      }
      steps {
        echo "Deploying"
      }
    }

    stage("Deploy to Production") {
      agent {
        label "lead-toolchain-skaffold"
      }
      when {
          branch 'master'
      }
      environment {
        TILLER_NAMESPACE = "${env.productionNamespace}"
        ISTIO_DOMAIN = "${env.productionDomain}"
        PRODUCT_NAME = "${env.product}"
      }
      steps {
        container('skaffold') {
          unstash 'build'
          sh "skaffold deploy -a image.json -n ${TILLER_NAMESPACE}"
          stageMessage "Successfully deployed to production:\nspringtrader-${env.product}.${env.productionDomain}/spring-nanotrader-web/"
        }
      }
    }


  }
}