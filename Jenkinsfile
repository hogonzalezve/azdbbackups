pipeline {
    agent any

    environment {
        AZURE_CREDENTIALS = credentials('azure-credentials-id')
    }

    stages {
        stage('Login to Azure') {
            steps {
                script {
                    sh 'az login --service-principal -u $AZURE_CREDENTIALS_USR -p $AZURE_CREDENTIALS_PSW --tenant $AZURE_CREDENTIALS_TENANT'
                }
            }
        }

        stage('Backup VM') {
            steps {
                script {
                    sh 'az backup protection backup-now --resource-group <resource-group> --vault-name <vault-name> --container-name <container-name> --item-name <vm-name> --backup-management-type AzureIaasVM --workload-type VM'
                }
            }
        }

        stage('Backup FileShare') {
            steps {
                script {
                    sh 'az backup protection backup-now --resource-group <resource-group> --vault-name <vault-name> --container-name <container-name> --item-name <fileshare-name> --backup-management-type AzureStorage --workload-type AzureFileShare'
                }
            }
        }
    }

    post {
        always {
            script {
                sh 'az logout'
            }
        }
    }
}
