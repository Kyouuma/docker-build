
pipeline {
    environment {
        DEPLOY = "${env.BRANCH_NAME == "master" || env.BRANCH_NAME == "develop" ? "true" : "false"}"
        NAME = "${env.BRANCH_NAME == "master" ? "example" : "docker-build-staging"}"
    }
    agent {
        kubernetes {
            inheritFrom "nodePodTemplate"
            yamlFile 'build-pod.yaml'
            defaultContainer 'node'
        }
    }

    options { 
        disableConcurrentBuilds() 
        buildDiscarder(logRotator(daysToKeepStr: '10', numToKeepStr: '5', artifactNumToKeepStr: '1'))
    }

    stages {
        stage('Make Image') {
            environment {
                PATH        = "/busybox:$PATH"
                REGISTRY    = 'acravaxia.azurecr.io' 
                REPOSITORY  = 'docker-build'
                IMAGE       = 'master'
            }
            steps {
                container(name: 'kaniko', shell: '/busybox/sh') {
                    ansiColor('xterm') {
                        sh '''#!/busybox/sh
                        /kaniko/executor -f `pwd`/Dockerfile -c `pwd` --cache=true --destination=${REGISTRY}/${REPOSITORY}:${IMAGE}
                        '''
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
                    withKubeConfig([credentialsId: 'jenkins-robot', serverUrl: 'https://dev-aks-20bb10c8.hcp.francecentral.azmk8s.io/']) {
                       sh '''
                           helm -n dev upgrade -i docker-build ./charts/docker-build/ \
                            --set image.tag=master \
                            --set ingress.enabled=true \
                            --set ingress.hosts[0].host=masterdev-20-74-10-207.nip.io \
                            --set ingress.tls[0].hosts[0]=masterdev-20-74-10-207.nip.io \
                            --set ingress.hosts[0].paths[0].path=/ \
                            --set ingress.tls[0].secretName=master-56-tls 
                       '''
                              }
                }
            }
        }
    }
}
