pipeline {
    agent any
    
     environment {
        VERSION = "3.4.0-SNAPSHOT"
    }

    stages {
        stage('notify') {
            steps {
                mail bcc: '', 
                 body: """Hi Team,

            Job Name: ${env.JOB_NAME}
            
            Regards,
            Jenkins
            """, 
             cc: '', from: '', replyTo: '', 
             subject: "[Jenkins] ${env.JOB_NAME} is started", 
             to: 'a23489200@gmail.com'
            }
        }
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/aadi5546/spring-petclinic-k8.git'
            }
        }
        stage('Maven Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv(installationName:'Sonarqube', credentialsId: 'sonar') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage('Maven Package') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Nexus Uploader') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'spring-petclinic', classifier: '', file: 'target/spring-petclinic-3.4.0-SNAPSHOT.jar', type: '.jar']], credentialsId: 'nexus', groupId: 'org.springframework.samples', nexusUrl: '34.203.237.221:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'spring-petclinic', version: '$VERSION'
            }
        }
        stage('Docker Image Build') {
            steps {
                sh "docker build -t aadii10/spring-petclinic-k8s:$VERSION ."
            }
        }
        
        stage('Docker Image Push') {
            steps {
                sh "docker push aadii10/spring-petclinic-k8s:$VERSION"
            }
        }
        stage('Kube deploy') {
            
            steps {
                sh "aws eks --region us-east-1 update-kubeconfig --name demo-cluster"
                sh "helm upgrade --install petclinic ./petclinic-helm"
            }
        }
    }
    post {
        success {
            mail bcc: '', 
                 body: """Hi Team,

            Job Name: ${env.JOB_NAME}
            Build Number: ${env.BUILD_NUMBER}
            Status: SUCCESS
            Branch: ${env.GIT_BRANCH}
            Commit: ${env.GIT_COMMIT}
            
            Logs: ${env.BUILD_URL}console
            Artifacts: ${env.BUILD_URL}artifact/
            
            Regards,
            Jenkins
            """, 
             cc: '', from: '', replyTo: '', 
             subject: "[Jenkins] ${env.JOB_NAME} #${env.BUILD_NUMBER} - Success", 
             to: 'a23489200@gmail.com'
        }
        failure {
            mail bcc: '', 
                 body: """Hi Team,

                Job Name: ${env.JOB_NAME}
                Build Number: ${env.BUILD_NUMBER}
                Status: FAILED
                Branch: ${env.GIT_BRANCH}
                Commit: ${env.GIT_COMMIT}
                
                Logs: ${env.BUILD_URL}console
                Artifacts: ${env.BUILD_URL}artifact/
                
                Regards,
                Jenkins
                """, 
                 cc: '', from: '', replyTo: '', 
                 subject: "[Jenkins] ${env.JOB_NAME} #${env.BUILD_NUMBER} - Failed", 
                 to: 'a23489200@gmail.com'
        }
    }
}
