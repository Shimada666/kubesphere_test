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
          git(url: 'git@github.com:Shimada666/kubesphere_test.git', changelog: true, poll: false, credentialsId: 'pz-github-ssh', branch: 'master')
        }

      }
    }

    stage('build & push') {
      steps {
        container('base') {
          sh 'docker build -f Dockerfile -t $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$GIT_COMMIT .'
          withCredentials([usernamePassword(credentialsId : 'txcloud-docker-registry' ,passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,)]) {
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
            sh '''envsubst < deployments/deployment.yml | kubectl apply -f -
envsubst < deployments/service.yml | kubectl apply -f -
envsubst < deployments/ingress.yml | kubectl apply -f -'''
          }

        }

      }
    }

  }
  environment {
    KUBECONFIG_CREDENTIAL_ID = 'kubeconfig'
    REGISTRY = 'ccr.ccs.tencentyun.com'
    DOCKERHUB_NAMESPACE = 'corgi_project'
    APP_NAME = 'ks-test'
  }
  parameters {
    string(name: 'TAG_NAME', defaultValue: '', description: '')
  }
}
