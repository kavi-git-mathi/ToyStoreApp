pipeline {
    agent any
    
    stages {
        stage('Git Checkout') {
            steps {
                echo "📥 Stage 1: Checking out code..."
                checkout scm
                echo "✅ Stage 1: Git checkout completed"
            }
        }
        
        // STAGE 2: .NET Build
        stage('.NET Build') {
            steps {
                echo "🔨 Stage 2: Building application..."
                sh '''
                    dotnet restore
                    dotnet build --configuration Release
                    echo "✅ Stage 2: .NET build completed"
                '''
            }
        }
    }
}