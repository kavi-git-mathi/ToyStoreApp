pipeline {
    agent any
    
    environment {
        APP_NAME = 'bloom-haven-nursery'
        ACR_REGISTRY = 'kavitharc.azurecr.io'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                checkout scm
                echo "✅ Stage 1: Git checkout completed"
            }
        }
    }
}