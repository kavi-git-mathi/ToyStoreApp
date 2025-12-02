stage('SonarQube Analysis') {
    steps {
        echo "📊 Stage 4: Running SonarQube analysis..."
        
        script {
            // Install SonarScanner for .NET
            sh '''
                echo "Installing SonarScanner for .NET..."
                dotnet tool install --global dotnet-sonarscanner --version 5.13.0 || true
            '''
            
            // Using SonarQube credentials from Jenkins
            withCredentials([usernamePassword(
                credentialsId: 'Sonarqube-token', // Replace with your credential ID
                usernameVariable: 'SONAR_USER',
                passwordVariable: 'SONAR_PASS'
            )]) {
                // Run SonarQube analysis
                withSonarQubeEnv('sonarqube') {
                    sh '''
                        echo "Starting SonarQube scan..."
                        dotnet sonarscanner begin \
                          /k:"${SONAR_PROJECT_KEY}" \
                          /d:sonar.host.url="http://localhost:9000" \
                          /d:sonar.login="${SONAR_USER}" \
                          /d:sonar.password="${SONAR_PASS}" \
                          /d:sonar.cs.vstest.reportsPaths="**/TestResults/*.trx"
                        
                        dotnet build --configuration Release
                        
                        dotnet sonarscanner end /d:sonar.login="${SONAR_USER}" /d:sonar.password="${SONAR_PASS}"
                        
                        echo "✅ SonarQube scan submitted"
                    '''
                }
            }
        }
    }
    
    post {
        success {
            echo "Waiting for SonarQube Quality Gate..."
            script {
                // Wait for quality gate result
                timeout(time: 5, unit: 'MINUTES') {
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "❌ SonarQube Quality Gate failed: ${qg.status}"
                    }
                    echo "✅ SonarQube Quality Gate passed: ${qg.status}"
                }
            }
        }
    }
}