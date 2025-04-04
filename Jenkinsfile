pipeline {
    agent any

    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal'
        RESOURCE_GROUP = 'rg-jenkins'
        APP_SERVICE_NAME = 'webapijenkinsdushyant'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Dushyant-1/python-pipeline.git'
            }
        }
         stage('Install Dependencies') {
            steps {
                sh 'pip install -r requirements.txt'
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
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat '''
                    if exist publish (rmdir /s /q publish)
                    mkdir publish
                    
                    :: Copy Python files and dependencies
                    xcopy /Y *.py publish\\
                    if exist requirements.txt copy requirements.txt publish\\
                    '''

                    bat 'az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%'
                    bat 'powershell Compress-Archive -Path ./publish/* -DestinationPath ./publish.zip -Force'
                    bat 'az webapp deploy --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --src-path ./publish.zip --type zip'
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
