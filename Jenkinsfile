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
        stage('upload the artifact'){
            steps{
                nexusArtifactUploader artifacts: [[artifactId: 'devops-integration', classifier: '', file: 'target/devops-integration.jar', type: 'jar']], credentialsId: 'Nexus-credential', groupId: 'com.truelearning', nexusUrl: '172.31.31.189:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'endproject/', version: '0.0.1-SNAPSHOT'
            }
        }    
        stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://172.31.92.131:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'cd /var/lib/jenkins/workspace/k8s-project && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
      stage('Build docker image'){
            steps{
                script{ 
                    sh 'ssh ubuntu@172.31.32.105'
                    sh 'scp /var/lib/jenkins/workspace/k8s-project/ ubuntu@172.31.32.105:/home/ubuntu'
                    sh 'docker build -t lokil5762049/pphproject:v4 .'
                }
            }
        }
          stage('Docker login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-pwd-loki', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh 'docker push lokil5762049/pphproject:v4'
                }
            }
        }
           stage('Deploying App to Kubernetes') {
      steps {
        script {
            sh 'ssh ubuntu@172.31.90.205'
          kubernetesDeploy(configs: "deploymentservice.yaml", kubeconfigId: "kubernetes-loki")               
                }
            }
        }
    }
}
