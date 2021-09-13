pipeline {
    environment {
        DEPLOY = "${env.BRANCH_NAME == "main" || env.BRANCH_NAME == "dev" ? "true" : "false"}"
        NAME = "${env.BRANCH_NAME == "main" ? "product-api" : "product-api-staging"}"
        BUILD_NUMBER = "${env.BUILD_NUMBER}"
        VERSION = "${env.BRANCH_NAME == "main" ? "BUILD_NUMBER" : "stg" + "-" + BUILD_NUMBER}"
        DOMAIN = 'localhost'
        REGISTRY = "${env.BRANCH_NAME == "main" ? 'rapixar/krea8-product-api' : "rapixar/stg-krea8-product-api"}"
        REGISTRY_CREDENTIAL = 'dockerhub-rapixar'
        NAMESPACE = "${env.BRANCH_NAME == "main" ? "krea8" : "stg-krea8"}"
        HELM_FILE = "${env.BRANCH_NAME == "main" ? "values.yaml" : "values-staging.yaml"}"
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
                    sh "helm upgrade --install ${NAME} --set ./helm -f ./helm/${HELM_FILE} --set name=${NAME} --set image.tag=${VERSION} -n=${NAMESPACE} --debug"
                }
            }
        }
    }
}