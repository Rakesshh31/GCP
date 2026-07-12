pipeline {
    agent any
	tools {
	jdk 'java17015'
	maven 'maven387'
    }

    environment {
	IMAGE_NAME = "java-app"
    IMAGE_TAG = "${BUILD_NUMBER}"
	ACR_NAME = "javarepooooooooooooooooooooo"
	ACR_LOGIN_SERVER = "${ACR_NAME}.azurecr.io"
	RESOURCE_GROUP = "1-7e01f143-playground-sandbox"
	ACI_NAME = "java-app-container"
	ACI_REGION = "eastus"
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

        stage('Authenticate with Azure'){
            steps {
                withCredentials([file(credentialsId: '', variable: 'AZURE_CRED')]) {
    // Use the AZURE_CRED variable to authenticate with Azure CLI
    sh '''
            echo 'Authenticating with Azure ...'
            az login --service-principal --username $(jq -r .clientId $AZURE_CRED) \
                             --password $(jq -r .clientSecret $AZURE_CRED) \
                             --tenant $(jq -r .tenantId $AZURE_CRED)' > /dev/null
            az account set --subscription $(jq -r .subscriptionId $AZURE_CRED)
    '''                 
                
            }
        }
    }
    
    stage('Tag & Push Image to Azure Container Registry (ACR)') {
            steps {
		withCredentials([file(credentialsId: 'azServicePrincipal', variable: 'AZURE_CRED')]) {
			sh '''
				echo 'Tagging and Pushing Image to ACR'
    				az acr login --name $ACR_NAME

          			echo "Tagging Docker image..."
          			docker tag $IMAGE_NAME:$IMAGE_TAG $ACR_LOGIN_SERVER/$IMAGE_NAME:$IMAGE_TAG

          			echo "Pushing Docker image to ACR..."
          			docker push $ACR_LOGIN_SERVER/$IMAGE_NAME:$IMAGE_TAG
       			'''
		}
            }
        }
	stage('Deploy to Azure Container Instance (ACI) & Get App URL') {
	    steps {
		withCredentials([file(credentialsId: 'azServicePrincipal', variable: 'AZURE_CRED')]) {
			sh '''
				echo 'Deploying Image to ACI'
				az container create \
            					--name $ACI_NAME \
            					--resource-group $RESOURCE_GROUP \
            					--image $ACR_LOGIN_SERVER/$IMAGE_NAME:$IMAGE_TAG \
            					--registry-login-server $ACR_LOGIN_SERVER \
            					--registry-username $(az acr credential show --name $ACR_NAME --query username -o tsv) \
            					--registry-password $(az acr credential show --name $ACR_NAME --query passwords[0].value -o tsv) \
            					--dns-name-label java-app-${BUILD_NUMBER} \
            					--ports 8090 \
            					--location $ACI_REGION \
		 				--os-type Linux \
       						--cpu 1 \
  						--memory 1.5 \
		 				--restart-policy Never

       					echo "Waiting for ACI to initialize..."
          				sleep 30

       					echo "Getting the URL of the ACI app..."
          				APP_URL=$(az container show \
            					--resource-group $RESOURCE_GROUP \
            					--name $ACI_NAME \
            					--query ipAddress.fqdn \
            					--output tsv):8090

          				echo "Application URL: http://$APP_URL"
    			'''
			}
		    }
	    }
	}
}
