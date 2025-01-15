pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: buildah
    image: quay.io/buildah/stable:latest
    securityContext:
      privileged: true
    tty: true
    command:
    - sleep
    args:
    - infinity
  - name: trivy
    image: aquasec/trivy:latest
    tty: true
    command:
    - sleep
    args:
    - infinity
'''
        }
    }
    environment {
        IMAGE_NAME = "ldonleycb/yokai-http"
        TAR_FILE = "image-${BUILD_NUMBER}.tar"
        REGISTRY = "docker.io"
        IMAGE_TAG = "${GIT_COMMIT[0..7]}"
    }
    stages {
        stage('checkout') {
            steps {
                checkout scm
            }
        }
        stage('build container') {
            steps {
                container('buildah') {
                    sh """
                    buildah build -t ${env.IMAGE_NAME}:${env.IMAGE_TAG} .
                    buildah push ${env.IMAGE_NAME}:${env.IMAGE_TAG} docker-archive:./${env.TAR_FILE}
                    """
                }
            }
        }
        stage('scan container') {
            steps {
                container('trivy') {
                    sh """
                    trivy image --input ${env.TAR_FILE} \
                        --format table
                    """
                }
            }
        }
        stage('Push image') {
            steps {
                container('buildah') {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'REGISTRY_USER',
                        passwordVariable: 'REGISTRY_PASS'
                    )]) {
                    sh """
                        buildah pull docker-archive:./${env.TAR_FILE}
                        buildah tag ${env.IMAGE_NAME}:${env.IMAGE_TAG} ${env.REGISTRY}/${env.IMAGE_NAME}:${env.IMAGE_TAG}
                        buildah tag ${env.IMAGE_NAME}:${env.IMAGE_TAG} ${env.REGISTRY}/${env.IMAGE_NAME}:latest
                        buildah login -u ${REGISTRY_USER} -p ${REGISTRY_PASS} ${env.REGISTRY}
                        buildah push ${env.REGISTRY}/${env.IMAGE_NAME}:${env.IMAGE_TAG}
                        buildah push ${env.REGISTRY}/${env.IMAGE_NAME}:latest
                    """
                    }
                }
            }
        }
    }
}
