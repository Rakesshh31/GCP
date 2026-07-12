pipeline {
    agent any

    stages {
        stage('Test Basic Pipeline') {
            steps {
                echo 'Pipeline is running'
            }
        }

        stage('Checkout Repository') {
            steps {
                echo 'Checking out repository code'
                checkout scm
            }
        }
    }
}