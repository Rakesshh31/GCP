pipeline {
    agent any
	tools {
	jdk 'java17015'
	maven 'maven387'
    }

    environment {
	SONAR_SCANNER_HOME = tool 'sonar7'
	IMAGE_NAME = "java-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
	GCP_PROJECT_ID = "focal-dock-440200-u5"
	FULL_IMAGE_NAME = "us-docker.pkg.dev/${GCP_PROJECT_ID}/java-app-repo-02/${IMAGE_NAME}:${IMAGE_TAG}"
	SERVICE_NAME = "java-app-service"
	REGION = "us-central1"
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
        stage('Trivy FS Scan'){
            steps {
                echo 'Scanning File System with Trivy FS ...'
		sh 'trivy fs --format table -o FSScanReport.html'
            }
        }

        stage('Build & Tag Docker Image'){
            steps {
                echo 'Building the Java App Docker Image'
		script {
			sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
		}
            }
        }
    }
}