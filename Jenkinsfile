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
        stage('Publish to Nexus') {
            steps{
              nexusArtifactUploader artifacts: [[artifactId: 'devops-integration', classifier: '', file: 'target/devops-integration.jar', type: 'jar']], credentialsId: 'nexus3', groupId: 'com.truelearning', nexusUrl: '172.31.86.15:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'endproject/', version: '0.0.1-SNAPSHOT' 
           }
        }
          stage('Build docker image'){
            steps{
                    script{
                       sshagent(['sshkeypair']) {
                       sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.87.128"
                       sh 'scp -i april-test.pem -r /var/lib/jenkins/workspace/k8s-project/ ubuntu@172.31.87.128:/home/ubuntu/k8s-project'
                       sh 'docker build -t lokil5762049/pphproject:v4 .'
                }
            }
        }
         }
        stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://172.31.95.62:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'cd /var/lib/jenkins/workspace/k8s-project && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
        stage('Docker login') {
            steps {
                sshagent(['sshkeypair']) {
                sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.87.128"
                withCredentials([usernamePassword(credentialsId: 'dockerhub-pwd-loki', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh 'docker push lokil5762049/pphproject:v4'
                }
            }
        }
          }
       stage('Deploying App to Kubernetes') {
      steps {
        script {
            sshagent(['sshkeypair']) {
                       sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.83.200"
                sh 'scp -i april-test.pem  /var/lib/jenkins/workspace/k8s-project/deploymentservice.yaml  ubuntu@172.31.83.200:/home/ubuntu/'
          kubernetesDeploy(configs: "deploymentservice.yaml", kubeconfigId: "kubernetes-loki")               
                }
            }
        }
    }
}
}
