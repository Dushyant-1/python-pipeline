


pipeline {
    agent any

    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal'  // Ensure this matches the ID in Jenkins credentials
        RESOURCE_GROUP = 'rg-jenkins'
        APP_SERVICE_NAME = 'webapijenkinsdushyant'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Dushyant-1/python-pipeline.git'
            }
        }

        stage('Set Up Python Environment') {
            steps {
                bat '"C:\\Program Files\\Python313\\python.exe" -m venv venv'
                bat '.\\venv\\Scripts\\activate && .\\venv\\Scripts\\python.exe -m pip install --upgrade pip'
                bat '.\\venv\\Scripts\\activate && .\\venv\\Scripts\\python.exe -m pip install -r requirements.txt'
                bat '.\\venv\\Scripts\\activate && .\\venv\\Scripts\\python.exe -m pip install pytest'
            }
        }

        // Optional: Uncomment this to run tests
        // stage('Run Tests') {
        //     steps {
        //         bat '.\\venv\\Scripts\\activate && .\\venv\\Scripts\\python.exe -m pytest'
        //     }
        // }

        stage('Deploy') {
            steps {
                withCredentials([azureServicePrincipal(
                    credentialsId: "${AZURE_CREDENTIALS_ID}",
                    clientIdVariable: 'AZURE_CLIENT_ID',
                    clientSecretVariable: 'AZURE_CLIENT_SECRET',
                    tenantIdVariable: 'AZURE_TENANT_ID',
                    subscriptionIdVariable: 'AZURE_SUBSCRIPTION_ID'
                )]) {
                    bat '''
                    if exist publish (rmdir /s /q publish)
                    mkdir publish

                    :: Copy Python files and requirements.txt to publish folder
                    for %%f in (*.py) do copy "%%f" publish\\
                    if exist requirements.txt copy requirements.txt publish\\

                    :: Login to Azure
                    az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%

                    :: Zip the publish folder
                    powershell Compress-Archive -Path ./publish/* -DestinationPath ./publish.zip -Force

                    :: Deploy to Azure App Service
                    az webapp deploy --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --src-path ./publish.zip --type zip

                    :: Optional: Logout from Azure
                    az logout
                    '''
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
