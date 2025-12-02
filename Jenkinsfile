pipeline {
    agent any
    
    environment {
        APP_NAME = 'toystoreapp'
        ACR_REGISTRY = 'kavitharc.azurecr.io'
    }
    
    stages {
        // STAGE 1: Already working
        stage('Git Checkout') {
            steps {
                checkout scm
                echo "✅ Stage 1: Git checkout completed"
            }
        }
        
        // STAGE 2: Already working
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
        
        // STAGE 3: Already working
        stage('.NET Tests') {
            steps {
                echo "🧪 Stage 3: Running tests..."
                sh '''
                    dotnet test --configuration Release
                    echo "✅ Stage 3: Tests completed"
                '''
            }
        }
        
        // STAGE 4: Trivy Security Scan (NEW)
        stage('Trivy Security Scan') {
            steps {
                echo "🛡️ Stage 4: Running security scan..."
                sh '''
                    # Install Trivy
                    curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin
                    
                    # Scan for vulnerabilities
                    trivy fs --severity HIGH,CRITICAL .
                    
                    echo "✅ Stage 4: Security scan completed"
                '''
            }
        }
        
        // We'll add Docker and ACR stages next
    }
}