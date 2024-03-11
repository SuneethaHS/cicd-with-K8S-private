pipeline {
    agent any
    environment {
        KUBECONFIG = '/var/lib/jenkins/workspace/k8s/' // Specify the path to your Kubernetes configuration file
    }
    stages{
        stage('Build Maven'){
            steps{
               git url:'https://github.com/SuneethaHS/cicd-with-K8S-private/', branch: "master"
               sh 'mvn clean install'
            }
        }
        stage('Publish to Nexus') {
            steps{
                nexusArtifactUploader artifacts: [[artifactId: 'devops-integration', classifier: '', file: 'target/devops-integration.jar', type: 'jar']], credentialsId: 'Nexus-credential', groupId: 'com.truelearning', nexusUrl: '172.31.86.15:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'endproject/', version: '0.0.1-SNAPSHOT'
                nexusArtifactUploader artifacts: [[artifactId: 'devops-integration', classifier: '', file: 'target/devops-integration.jar', type: 'jar']], credentialsId: '0d013729-71f6-4223-a909-b3be4e1453bf', groupId: 'com.truelearning', nexusUrl: '172.31.43.37:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'endproject', version: '0.0.1-SNAPSHOT'
           }
        }
          stage('Build docker image'){
            steps{
                script{
                   sh 'docker build -t suneethahs/private:v1 .'
                }
          }
        }
        stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://13.233.166.222:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'cd /var/lib/jenkins/workspace/k8s-project && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
        stage('Docker login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh 'docker push suneethahs/private:v1'
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
