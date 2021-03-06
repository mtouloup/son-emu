pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                echo 'Checkout...'
                checkout scm
            }
        }
        stage('Build') {
            steps {
                echo 'Building...'
                sh "docker build --no-cache -t sonatanfv/son-emu:dev ."
            }
        }
        stage('Style check') {
            steps {
                echo 'Style check...'
                sh "docker run --name son-emu --rm --privileged --pid='host' -v /var/run/docker.sock:/var/run/docker.sock sonatanfv/son-emu:dev 'flake8 --exclude=.eggs,devops --ignore=E501,W605,W504 .'"
                echo "done."
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
                sh "docker run --name son-emu --rm --privileged --pid='host' -v /var/run/docker.sock:/var/run/docker.sock sonatanfv/son-emu:dev 'pytest -v'"
            }
        }
        stage('Package') {
            steps {
                echo 'Packaging (Docker-image)...'
                // push to public Docker registry
                sh "docker push sonatanfv/son-emu:dev"
                // might be moved to another job (:dev and :latest are the same for now)
                sh "docker tag sonatanfv/son-emu:dev sonatanfv/son-emu:latest"
                sh "docker push sonatanfv/son-emu:latest"
                // push to internal Docker registry
                sh "docker tag sonatanfv/son-emu:dev registry.sonata-nfv.eu:5000/son-emu:latest"
                sh "docker push registry.sonata-nfv.eu:5000/son-emu"        
            }
        }
    }
    post {
         success {
                 emailext(from: "jenkins@sonata-nfv.eu", 
                 to: "manuel.peuster@upb.de", 
                 subject: "SUCCESS: ${env.JOB_NAME}/${env.BUILD_ID} (${env.BRANCH_NAME})",
                 body: "${env.JOB_URL}")
         }
         failure {
                 emailext(from: "jenkins@sonata-nfv.eu", 
                 to: "manuel.peuster@upb.de", 
                 subject: "FAILURE: ${env.JOB_NAME}/${env.BUILD_ID} (${env.BRANCH_NAME})",
                 body: "${env.JOB_URL}")
         }
    }
}

