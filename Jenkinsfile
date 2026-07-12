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

        stage('Checkout Repository') {
            steps {
                echo 'Checking out repository code'
                checkout scm
            }
        }

        stage('Build with Maven') {
            steps {
                echo 'Building Java project with Maven'
                sh 'mvn clean package'
            }
        }
    }
}