pipeline {
    agent any
	tools {
	jdk 'java17015'
	maven 'maven387'
    }

    environment {
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
                cleanWs()
                echo 'Checking out GitHub Codes ...'
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'jenkins-gcp', url: 'https://github.com/Rakesshh31/GCP.git']])
                // Ensure workspace matches origin/main exactly (avoid stale files)
                sh 'git fetch --all'
                sh 'git reset --hard origin/main'
                sh 'git clean -fdx'
                // Print current commit and the Dockerfile to verify pipeline uses the expected files
                sh 'git rev-parse --short HEAD'
                sh 'sed -n "1,40p" Dockerfile'
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

        stage('Debug Dockerfile') {
           steps {
        sh '''
        pwd
        ls -la
        echo "========== Dockerfile =========="
        cat Dockerfile
        '''
    }
}

        stage('Build & Tag Docker Image'){
            steps {
                echo 'Building the Java App Docker Image'
		script {
			// build without cache to avoid using a stale base image reference
			sh "docker build --no-cache -t ${IMAGE_NAME}:${IMAGE_TAG} ."
		}
            }
        }

        stage('Trivy Security Scan'){
            steps {
                echo 'Scanning Docker Image with Trivy'
		sh '''
  			trivy --severity HIGH,CRITICAL --cache-dir ${WORKSPACE}/.trivy-cache --no-progress --format table -o trivyFSScanReport.html image ${IMAGE_NAME}:${IMAGE_TAG}
     		'''
            }
        }
    }
}