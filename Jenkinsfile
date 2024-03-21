pipeline {
    agent any
    environment {
        KUBECONFIG = '/var/lib/jenkins/workspace/k8s/' // Specify the path to your Kubernetes configuration file
    }
    stages{
        stage('code'){
            steps{
               git url:'https://github.com/SuneethaHS/cicd-with-K8S-private/', branch: "master"
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(credentialsId: 'sonarqube', installationName: 'Sonarqube') {
                    sh "mvn clean verify sonar:sonar -Dsonar.projectKey=cicd"
                }
            }
        }
        stage("Quality gate"){
            steps{
              waitForQualityGate abortPipeline: true
            }
            }
        stage('Build Maven'){
            steps{
               sh 'mvn clean install'
            }
        }
        stage('Publish to Nexus') {
            steps{
               nexusArtifactUploader artifacts: [[artifactId: 'devops-integration', classifier: '', file: 'target/devops-integration.jar', type: 'jar']], credentialsId: 'nexus', groupId: 'com.truelearning', nexusUrl: '172.31.43.37:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'endproject', version: '0.0.1-SNAPSHOT'
           }
        }
          stage('Build docker image'){
            steps{
                script{
                   sh 'docker build -t suneethahs/private:v2 .'
                }
          }
        }
        stage('Docker login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh 'docker push suneethahs/private:v2'
            }
        }
        }    
       stage('Deploying App to Kubernetes') {
      steps {
        script {
             kubernetesDeploy(configs: "deploymentservice.yaml", kubeconfigId: "kubernetes-suni")               
                }
            }
        }
    }
}
