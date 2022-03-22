pipeline {
  agent {
    kubernetes {
      defaultContainer 'jnlp'
      yaml '''
      apiVersion: v1
      kind: Pod
      metadata:
      spec:
        containers:
        - name: node
          image: node:lts-alpine
          imagePullPolicy: IfNotPresent
          command:
          - cat
          tty: true
        - name: 
      '''
    }
  }

  environment {
    Version_Major = 1
    Version_Minor = 0
    Version_Patch = 0
    IMAGE_NAME = "10.52.51.109:5000/carikatla"
    IMAGE_TAG = "${Version_Major}.${Version_Minor}.${Version_Patch}-${BUILD_TIMESTAMP}-${env.BUILD_NUMBER}"
    CLUSTER_CONTEXT = credentials('cluster-context')
    CLUSTER_USER = credentials('cluster-user')
    CI = false
  }

  stages {
    stage('Build') {
      steps {
        container('node') {
          sh """
            echo "*** building ***"
            CI=${CI}
            npm install
            npm run build
          """
        }
      }
    }
    stage('Push Image') {
      steps {
        container('docker') {
          sh """
            echo "*** pushing image ***"
            docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
            docker push ${IMAGE_NAME}:${IMAGE_TAG}
          """
        }
      }
    }
    stage('Helm') {
      steps {
        sh """
          echo "*** deploying ***"
          kubectl config set-context ${CLUSTER_CONTEXT} --cluster=kubernetes --user=${CLUSTER_USER}
          kubectl config use-context ${CLUSTER_CONTEXT}
          helm upgrade -i carikatla helm/carikatla -f helm/carikatla/values.yaml -n carikatla --set=image.tag=${IMAGE_TAG} --create-namespace
          kubectl rollout status deployment/carikatla -n carikatla
          kubectl get pods -n carikatla
          helm ls -n carikatla
        """
      }
    }
  }
}
