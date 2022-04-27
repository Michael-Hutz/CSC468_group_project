pipeline {
    agent none 
    environment {
        docker_user = "mhutz"
    }
    stages {
        stage('Publish') {
            agent {
                kubernetes {
                    inheritFrom 'agent-template'
                }
            }
            steps{
                container('docker') {
                    sh 'echo $DOCKER_TOKEN | docker login --username $DOCKER_USER --password-stdin'
                    sh 'cd rngtech; docker build -t $DOCKER_USER/rngtech:$BUILD_NUMBER .'
                    sh 'docker push $DOCKER_USER/rngtech:$BUILD_NUMBER'
                }
            }
        }
        stage ('Deploy') {
            agent {
                node {
                    label 'deploy'
                }
            }
            steps {
                sshagent(credentials: ['cloudlab']) {
                    sh "sed -i 's/DOCKER_REGISTRY/${docker_user}/g' rngtech.yaml"
                    sh "sed -i 's/BUILD_NUMBER/${BUILD_NUMBER}/g' rngtech.yaml"
                    sh 'scp -r -v -o StrictHostKeyChecking=no *.yaml MH824134@clnodevm063-1.clemson.cloudlab.us:~/'
                    sh 'ssh -o StrictHostKeyChecking=no MH824134@clnodevm063-1.clemson.cloudlab.us kubectl apply -f /users/MH824134/rngtech.yaml -n jenkins'
                    sh 'ssh -o StrictHostKeyChecking=no MH824134@clnodevm063-1.clemson.cloudlab.us kubectl apply -f /users/MH824134/rngtech-service.yaml -n jenkins'                                        
                }
            }
        }
    }
}