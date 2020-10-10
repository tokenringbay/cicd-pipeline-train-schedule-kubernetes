pipeline {
    agent any
    environment {
        //be sure to replace "tkoulech" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "tkoulech/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'example-solution'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'example-solution'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('SkipDeployToProduction') {
            when {
                branch 'non-example-solution'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
        stage('DeployToProduction') {
            when { 
                branch 'example-solution'
            }
            steps {
                script{
                   //def image_id = registry + ":$BUILD_NUMBER"
                   sh "/usr/local/bin/kubectl apply -f train-schedule-kube.yml.yml"
               }
            }
        }
    }
}
