
pipeline {
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
        
        stage('Make Image') {
            environment {
                PATH        = "/busybox:$PATH"
                REGISTRY    = 'acravaxia.azurecr.io' // Configure your own registry
                REPOSITORY  = 'docker-build'
                IMAGE       = 'master'
            }
            steps {
                container(name: 'kaniko', shell: '/busybox/sh') {
                    sh '''#!/busybox/sh
                    /kaniko/executor -f `pwd`/Dockerfile -c `pwd` --cache=true --destination=${REGISTRY}/${REPOSITORY}:${IMAGE}
                    '''
                }
            }
        }

      
    }
}
