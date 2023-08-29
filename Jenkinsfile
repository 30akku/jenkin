pipeline {
    agent any
    stages {
        stage('SCA') {
            agent
            {
                label 'windows'
            }            
            steps {
                echo "Steps to execute SCA"
                withSonarQubeEnv(installationName: 'SonarQube', credentialsId: 'SonarToken') {
                    bat 'C:\Users\AK C\Downloads\sonar-scanner-cli-5.0.1.3006-windows\\bin\\sonar-scanner -Dsonar.projectVersion=1.0 -Dsonar.projectKey=spring-app -Dsonar.sources=src -Dsonar.java.binaries=.'
                }
                waitForQualityGate(abortPipeline: true, credentialsId: 'SonarToken')
            }
        }
        stage('Unit Tests') {
            agent {
                docker {
                    image 'maven:latest'
                    args '--network host -v $HOME/.m2:/root/.m2'
                    reuseNode true
                }
            }
            steps {
                sh 'mvn -version'
                sh 'java -version'
                sh 'mvn test'
                junit '**/target/surefire-reports/TEST-*.xml'
                jacoco buildOverBuild: true, runAlways: true
            }
        }                
        stage('Build') {
            agent {
                docker {
                    image 'maven:latest'
                    args '--network host -v $HOME/.m2:/root/.m2'
                    reuseNode true
                }
            }                
            steps {
                sh 'mvn -version'
                sh 'java -version'
                sh 'mvn package -Dmaven.test.skip=true'
                archiveArtifacts artifacts: 'target/*.jar', followSymlinks: false
                stash(includes: 'target/*.jar', name: 'build')
            }
        }        
        stage('Docker Image') {
            steps {
                unstash 'build'
                sh 'docker info'
                sh 'ls -l'
                sh 'docker build -f Dockerfile -t mitesh51/ms-petclinic:1.0 .'
                sh 'docker images'
    
                withDockerRegistry([ credentialsId: "dockerhub-cred", url: "" ]) {
                    sh 'docker push agust30xyz/ms-petclinic:1.0'
                }
            }
        }
        stage('Trivy Image Scanner') {
            agent {
                docker { 
                    image 'aquasec/trivy:latest'
                    args '--network host --entrypoint='
                }
            }  
            options { skipDefaultCheckout() }
            steps {
                sh 'trivy help'
                sh "trivy --cache-dir /tmp i 'mitesh51/ms-petclinic:1.0'"
            }
        }
        stage('Approval') {
            steps {
                input 'Ready for Deployment?'
            }
        }
        stage('minikube-Deployment') {
            agent
            {
                label 'windows'
            }
            steps {
                bat 'kubectl apply -f petclinic-deployment.yaml'
            }
        }
    }
}