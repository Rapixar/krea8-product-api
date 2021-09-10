pipeline {
    environment {
        DEPLOY = "${env.BRANCH_NAME == "master" || env.BRANCH_NAME == "dev" ? "true" : "false"}"
        NAME = "${env.BRANCH_NAME == "master" ? "product-api" : "product-api-staging"}"
        BUILD_NUMBER = "${env.BUILD_NUMBER}"
        VERSION = "${env.BRANCH_NAME == "master" ? "BUILD_NUMBER" : "stg" + "-" + BUILD_NUMBER}"
        DOMAIN = 'localhost'
        REGISTRY = 'rapixar/krea8-product-api'
        REGISTRY_CREDENTIAL = 'dockerhub-rapixar'
        NAMESPACE = 'krea8'
    }
    agent {
        kubernetes {
            defaultContainer 'jnlp'
            yamlFile 'build.yaml'
        }
    }
    stages {
        stage('Build') {
            steps {
                container('golang') {
                    sh 'go get -d -v ./...'
                    sh 'go build -o /go/bin/app -v ./...'
                }
            }
        }
        stage('Docker Build') {
            when {
                environment name: 'DEPLOY', value: 'true'
            }
            steps {
                container('docker') {
                    sh "docker build -t ${REGISTRY}:${VERSION} ."
                }
            }
        }
        stage('Docker Publish') {
            when {
                environment name: 'DEPLOY', value: 'true'
            }
            steps {
                container('docker') {
                    withDockerRegistry([credentialsId: "${REGISTRY_CREDENTIAL}", url: ""]) {
                        sh "docker push ${REGISTRY}:${VERSION}"
                    }
                }
            }
        }
        stage('Kubernetes Deploy') {
            when {
                environment name: 'DEPLOY', value: 'true'
            }
            steps {
                container('helm') {
                    sh "helm upgrade --install --force --set name=${NAME} --set image.tag=${VERSION} --set domain=${DOMAIN} ${NAME} ./helm -n=${NAMESPACE}"
                }
            }
        }
    }
}