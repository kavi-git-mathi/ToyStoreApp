pipeline {
    agent any
    
    options {
        // Keep only last 3 builds to save space
        buildDiscarder(logRotator(numToKeepStr: '3'))
        
        // Timeout after 30 minutes
        timeout(time: 30, unit: 'MINUTES')
        
        // Skip default checkout (we'll do it manually)
        skipDefaultCheckout()
    }
    
    environment {
        APP_NAME = 'toystoreapp'
        ACR_REGISTRY = 'kavitharc.azurecr.io'
        DOCKER_IMAGE = "${ACR_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}"
    }
    
    stages {
        // STAGE 1: Clean Workspace & Git Checkout
        stage('Clean & Checkout') {
            steps {
                echo "🧹 Stage 1: Cleaning workspace..."
                
                script {
                    // Manual cleanup before checkout
                    sh '''
                        echo "Cleaning workspace..."
                        # Remove everything except hidden Jenkins files
                        find . -maxdepth 1 ! -name '.' ! -name '..' ! -name '*.jenkins*' -exec rm -rf {} + 2>/dev/null || true
                        
                        echo "Workspace cleaned. Current contents:"
                        ls -la
                    '''
                }
                
                echo "📥 Checking out code..."
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    extensions: [
                        // Clean before checkout
                        [$class: 'CleanBeforeCheckout'],
                        // Shallow clone to save space
                        [$class: 'CloneOption', depth: 1, shallow: true]
                    ],
                    userRemoteConfigs: [[
                        url: 'https://github.com/kavi-git-mathi/Toystoreapp.git'
                    ]]
                ])
                
                sh '''
                    echo "✅ Checkout completed"
                    echo "Workspace size:"
                    du -sh . || true
                '''
            }
        }
        
        // STAGE 2: .NET Build
        stage('.NET Build') {
            steps {
                echo "🔨 Stage 2: Building application..."
                sh '''
                    # Clean any previous build artifacts
                    rm -rf bin/ obj/ 2>/dev/null || true
                    
                    dotnet restore --verbosity minimal
                    dotnet build --configuration Release --no-restore --verbosity minimal
                    
                    echo "Build completed"
                    echo "Build output size:"
                    du -sh bin/ 2>/dev/null || true
                '''
            }
        }
        
        // STAGE 3: .NET Tests
        stage('.NET Tests') {
            steps {
                echo "🧪 Stage 3: Running tests..."
                sh '''
                    dotnet test --configuration Release --no-build --verbosity minimal
                    echo "✅ Tests completed"
                '''
            }
        }
        
        // STAGE 4: Trivy Security Scan
        stage('Security Scan') {
            steps {
                echo "🛡️ Stage 4: Security scanning..."
                sh '''
                    # Create temp directory for Trivy
                    TRIVY_TEMP="${WORKSPACE}/.trivy-temp"
                    mkdir -p ${TRIVY_TEMP}
                    
                    echo "Installing Trivy..."
                    curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b ${TRIVY_TEMP}
                    
                    echo "Running security scan..."
                    ${TRIVY_TEMP}/trivy fs --severity HIGH,CRITICAL --exit-code 0 --quiet .
                    
                    echo "✅ Security scan completed"
                    
                    # Clean Trivy temp files
                    rm -rf ${TRIVY_TEMP}
                '''
            }
        }
        
        // STAGE 5: Docker Build
        stage('Docker Build') {
            steps {
                echo "🐳 Stage 5: Building Docker image..."
                sh '''
                    echo "Cleaning Docker cache before build..."
                    docker image prune -f 2>/dev/null || true
                    
                    echo "Building Docker image..."
                    docker build \
                      --tag ${DOCKER_IMAGE} \
                      --tag ${ACR_REGISTRY}/${APP_NAME}:latest \
                      .
                    
                    echo "✅ Docker image built"
                    echo "Images created:"
                    docker images --filter "reference=*${APP_NAME}*" --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}" 2>/dev/null || true
                '''
            }
        }
        
        // STAGE 6: Push to ACR
        stage('Push to ACR') {
            steps {
                echo "🚀 Stage 6: Pushing to ACR..."
                
                withCredentials([usernamePassword(
                    credentialsId: 'acr-credentials',
                    usernameVariable: 'ACR_USER',
                    passwordVariable: 'ACR_PASS'
                )]) {
                    sh '''
                        # Login to ACR
                        docker login ${ACR_REGISTRY} -u $ACR_USER -p $ACR_PASS
                        
                        # Push images
                        docker push ${DOCKER_IMAGE}
                        docker push ${ACR_REGISTRY}/${APP_NAME}:latest
                        
                        echo "✅ Images pushed to ACR"
                        echo "- ${DOCKER_IMAGE}"
                        echo "- ${ACR_REGISTRY}/${APP_NAME}:latest"
                    '''
                }
            }
        }
    }
    
    post {
        always {
            echo "🧹 Performing final cleanup..."
            
            script {
                // Comprehensive cleanup
                sh '''
                    echo "=== CLEANING WORKSPACE ==="
                    # Remove build artifacts
                    rm -rf bin/ obj/ TestResults/ .trivy-temp/ 2>/dev/null || true
                    
                    # Clean Docker
                    echo "Cleaning Docker..."
                    docker rmi ${DOCKER_IMAGE} ${ACR_REGISTRY}/${APP_NAME}:latest 2>/dev/null || true
                    docker image prune -f 2>/dev/null || true
                    docker builder prune -f 2>/dev/null || true
                    
                    echo "=== FINAL DISK USAGE ==="
                    echo "Workspace size:"
                    du -sh . 2>/dev/null || echo "Cannot check workspace size"
                    echo "Docker disk usage:"
                    docker system df 2>/dev/null || echo "Cannot check Docker disk usage"
                '''
            }
        }
        
        success {
            echo "✅ Pipeline completed successfully with cleanup!"
        }
        
        failure {
            echo "❌ Pipeline failed - cleanup performed"
        }
    }
}