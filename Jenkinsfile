pipeline {
    agent any
	tools {
	jdk 'java17015'
	maven 'maven387'
    }
    stages {
        stage('Initialize Pipeline'){
            steps {
                echo 'Initializing Pipeline ...'
		sh 'java -version'
		sh 'mvn -version'
            }
        }

        stage('Checkout GitHub Codes'){
            steps {
                echo 'Checking out GitHub Codes ...'
		checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'jenkins-gcp', url: 'https://github.com/iQuantC/Jenkins_GCP_CloudRun.git']])
            }
        }

        stage('Maven Build'){
            steps {
                echo 'Building Java App with Maven'
		sh 'mvn clean package'
            }
        }
        stage('JUnit Test of Java App'){
            steps {
                echo 'JUnit Testing'
		sh 'mvn test'
            }
        }
    }
}