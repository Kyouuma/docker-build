
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
        stage('Build Docker Image') {
            environment {
                PATH        = "/busybox:$PATH"
                REGISTRY    = 'acravaxia.azurecr.io' 
                REPOSITORY  = 'docker-build'
            }
            steps {
                container(name: 'kaniko', shell: '/busybox/sh') {
                    ansiColor('xterm') {
                        sh '''#!/busybox/sh
                        /kaniko/executor -f `pwd`/Dockerfile -c `pwd` --cache=true --destination=${REGISTRY}/${REPOSITORY}:${BRANCH_NAME}
                        '''
                    }
                }
            }
        }
        stage('Deploy To Integration') {
            when {
                branch 'master'
            }
            steps {
                container('helm') {
                    withKubeConfig([credentialsId: 'jenkins-robot', serverUrl: 'https://dev-aks-20bb10c8.hcp.francecentral.azmk8s.io/']) {
                       sh '''
                           helm -n dev upgrade -i demo-int ./charts/docker-build/ \
                            --set image.tag=latest \
                            --set ingress.enabled=true \
                            --set ingress.hosts[0].host=demo-int-20-74-10-207.nip.io \
                            --set ingress.tls[0].hosts[0]=demo-int-20-74-10-207.nip.io \
                            --set ingress.hosts[0].paths[0].path=/ \
                            --set ingress.tls[0].secretName=demo-int-tls 
                       '''
                    }
                }
            }
        }
        stage('Deploy To Staging') {
            when {
                buildingTag()
            }
            steps {
                container('helm') {
                    withKubeConfig([credentialsId: 'jenkins-robot', serverUrl: 'https://dev-aks-20bb10c8.hcp.francecentral.azmk8s.io/']) {
                       sh '''
                           helm -n dev upgrade -i demo-staging ./charts/docker-build/ \
                            --set image.tag=${BRANCH_NAME} \
                            --set ingress.enabled=true \
                            --set ingress.hosts[0].host=demo-staging-20-74-10-207.nip.io \
                            --set ingress.tls[0].hosts[0]=demo-staging-20-74-10-207.nip.io \
                            --set ingress.hosts[0].paths[0].path=/ \
                            --set ingress.tls[0].secretName=demo-staging-tls 
                       '''
                    }
                }
            }
        }
    }
}
