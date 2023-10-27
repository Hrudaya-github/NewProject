pipeline {
    agent any  
    stages {
        stage('SCM'){
            steps {
                git branch: 'main', url: 'https://github.com/Hrudaya-github/jenkins-project.git'
            }
        }
        stage('Unit Testing'){
            steps {
                sh 'mvn test'
            }
        }
        stage('Integration Testing'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
        stage('Maven Build'){
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Build & Sonarqube Analysis'){
            steps {
                withSonarQubeEnv('sonar-server'){
                  sh script: 'mvn clean package sonar:sonar'
                }
            }                
        }
        stage('Quality Gate'){
            steps {
                timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true, credentialsId:'sonar-jenkins-auth'
                }
            }
        }
        stage('upload artifact in nexus'){
            steps {
                script {
                    def readPomVersion = readMavenPom file: 'pom.xml'
                    def nexusRepo = readPomVersion.version.endsWith("SNAPSHOT") ? "nexus-app-snapshot" : "nexus-app-release"
                    nexusArtifactUploader artifacts: 
                    [
                        [
                            artifactId: 'springboot', 
                            classifier: '', 
                            file: 'target/Uber.jar',
                             type: 'jar'
                        ]
                    ],
                    credentialsId: 'nexus-auth',
                    groupId: 'com.example',
                    nexusUrl: '44.242.165.161:8081',
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    repository: nexusRepo,
                    version: "${readPomVersion.version}"
                }
            }    
        }  
    }
}
