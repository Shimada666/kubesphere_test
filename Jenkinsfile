pipeline {
  agent {
    node {
      label 'base'
    }

  }
  stages {
    stage('clone code') {
      agent none
      steps {
        container('base') {
          git(url: "${REPO}", changelog: true, poll: false, credentialsId: "${GIT_CREDENTIAL_ID}", branch: 'master')
        }

      }
    }

    stage('build & push') {
      steps {
        container('base') {
          sh 'docker build -f Dockerfile -t $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$GIT_COMMIT .'
          withCredentials([usernamePassword(credentialsId : "${DOCKER_REGISTRY_CREDENTIAL_ID}" ,passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,)]) {
            sh 'echo "$DOCKER_PASSWORD" | docker login $REGISTRY -u "$DOCKER_USERNAME" --password-stdin'
            sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$GIT_COMMIT'
          }

        }

      }
    }

    stage('push latest') {
      steps {
        container('base') {
          sh 'docker tag  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$GIT_COMMIT $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest '
          sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest '
        }

      }
    }

    stage('deploy to production') {
      agent none
      steps {
        container('base') {
          withCredentials([kubeconfigContent(credentialsId : 'kubeconfig' ,variable : 'KUBECONFIG_CONFIG' ,)]) {
            sh 'mkdir -p ~/.kube/'
            sh 'echo "$KUBECONFIG_CONFIG" > ~/.kube/config'
            sh '''envsubst < deployments/deploy.yaml | kubectl apply -f -'''
          }

        }

      }
    }

  }
  environment {
    REPO = 'git@github.com:Shimada666/kubesphere_test.git'
    GIT_CREDENTIAL_ID = 'pz-github-ssh'
    KUBECONFIG_CREDENTIAL_ID = 'kubeconfig'
    REGISTRY = 'ccr.ccs.tencentyun.com'
    DOCKER_REGISTRY_CREDENTIAL_ID = 'txcloud-docker-registry'
    DOCKERHUB_NAMESPACE = 'corgi_project'
    APP_NAME = 'ks-test'
  }
  parameters {
    string(name: 'TAG_NAME', defaultValue: '', description: '')
  }
}
