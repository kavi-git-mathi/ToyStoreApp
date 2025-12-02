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
        
        // STAGE 3: .NET Tests
        stage('.NET Tests') {
            steps {
                echo "🧪 Stage 3: Running tests..."
                sh '''
                    dotnet test --configuration Release
                    echo "✅ Stage 3: Tests completed"
                '''
            }
        }
    }
}