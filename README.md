pipeline {
    agent any
    stages {
        stage ("Git Checkout") {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/rahulpatel2605/JenkinsProject-TCK.git']])
            }
        }
        
        stage ("docker image build") {
            steps {
                sh "docker build -t $JOB_NAME.$BUILD_ID ."
                sh "docker image tag $JOB_NAME.$BUILD_ID rahulp2605/$JOB_NAME.$BUILD_ID"
                sh "docker image tag rahulp2605/$JOB_NAME.$BUILD_ID rahulp2605/$JOB_NAME.$BUILD_ID:latest"
            }
        }
        
        stage ("push docker image to decker hub") {
            steps {
            withCredentials([string(credentialsId: 'rahulp2605', variable: 'DockerHubPassword')]) {
    // some block
    sh "docker login -u rahulp2605 -p ${DockerHubPassword}"
    sh "docker image push rahulp2605/$JOB_NAME.$BUILD_ID:latest"
}
            }
        }
        
        stage ("Deployment") {
            steps {
                sshagent(['jenkinshostpassword']) {
    // some block
    sh "docker container run -p 8000:80 -d rahulp2605/$JOB_NAME.${env.BUILD_ID}:latest"
}
            }
        }
    }
}
