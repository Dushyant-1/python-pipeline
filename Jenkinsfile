pipeline {
    agent any

    environment {
        RESOURCE_GROUP = 'rg-jenkins'
        APP_SERVICE_NAME = 'app-service-plan123'  // Fixed App Service name
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Dushyant-1/python-pipeline.git'
            }
        }

        stage('Set Up Python Environment') {
            steps {
                bat 'python -m venv venv'
                bat 'venv\\Scripts\\python.exe -m pip install --upgrade pip'
                bat 'venv\\Scripts\\python.exe -m pip install -r requirements.txt'
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([
                    string(credentialsId: 'azure-client-id', variable: 'AZURE_CLIENT_ID'),
                    string(credentialsId: 'azure-client-secret', variable: 'AZURE_CLIENT_SECRET'),
                    string(credentialsId: 'azure-tenant-id', variable: 'AZURE_TENANT_ID')
                ]) {
                    withEnv([
                        'AZURE_CLIENT_ID=%AZURE_CLIENT_ID%',
                        'AZURE_CLIENT_SECRET=%AZURE_CLIENT_SECRET%',
                        'AZURE_TENANT_ID=%AZURE_TENANT_ID%'
                    ]) {
                        bat '''
                        if exist publish (rmdir /s /q publish)
                        mkdir publish
                        
                        :: Copy Python files and dependencies
                        xcopy /Y *.py publish\\
                        if exist requirements.txt copy requirements.txt publish\\
                        '''

                        :: Ensure Azure CLI is installed and authenticated
                        bat 'az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%'
                        
                        :: Package and deploy to Azure
                        bat 'powershell Compress-Archive -Path ./publish/* -DestinationPath ./publish.zip -Force'
                        bat 'az webapp deploy --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --src-path ./publish.zip --type zip'
                    }
                }
            }
        }
    }

    post {
        failure {
            echo 'Deployment Failed!'
        }
        success {
            echo 'Deployment Successful!'
        }
    }
}
