pipeline {
    agent any
    environment {
        KUBECONFIG = '/var/lib/jenkins/workspace/k8s/' // Specify the path to your Kubernetes configuration file
    }
    stages{
        stage('Build Maven'){
            steps{
                git url:'https://github.com/LOKESH-creator660/cicd-with-K8S/', branch: "master"
               sh 'mvn clean install'
            }
        }
        stage('Build docker image'){
            steps{
                script{ 
                    sh 'docker build -t lokil5762049/endtoendproject:v1 .'
                }
            }
        }
          stage('Docker login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-pwd-loki', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh 'docker push lokil5762049/endtoendproject:v1'
                }
            }
        }
           stage('Deploying App to Kubernetes') {
      steps {
        script {
          kubernetesDeploy(configs: "deploymentservice.yaml", kubeconfigId: "kubernetes-loki")               
                }
            }
        }
    }
}
