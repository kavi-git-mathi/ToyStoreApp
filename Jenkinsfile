pipeline {
    agent any
    
    environment {
        APP_NAME = 'toystoreapp'
        ACR_REGISTRY = 'kavitharc.azurecr.io'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                checkout scm
                echo "✅ Stage 1: Git checkout completed"
            }
        }
        
        stage('.NET Restore & Build') {
            steps {
                echo "🔨 Stage 2: Building .NET application..."
                sh '''
                    dotnet restore
                    dotnet build --configuration Release
                    echo "✅ Stage 2: .NET build completed"
                '''
            }
        }
    }
}