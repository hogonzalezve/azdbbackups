#!/usr/bin/env groovy

def projectProperties = [
    buildDiscarder(logRotator(numToKeepStr: '10')),
]

properties(projectProperties)

// Definir Variables Globales
//@Field def AZURE_CREDENTIALS_ID = 'azure-credentials-id'

pipeline {
    agent any

    options {
        timeout(time: 30, unit: 'MINUTES')
    }

    // parameters {
    //     string(name: 'VM_LIST', defaultValue: '', description: 'Comma-separated list of VM names. Ref: vm1,vm2,vm3')
    //     string(name: 'ADD_VM_GROUP_START_TAG_KEY', defaultValue: '', description: 'Key for the start group tag. Ref: grupo1start, grupo2start ')
    //     string(name: 'ADD_VM_GROUP_STOP_TAG_KEY', defaultValue: '', description: 'Key for the stop group tag. Ref: grupo1stop, grupo2stop')
    //     string(name: 'REMOVE_VM_GROUP_START_TAG_KEY', defaultValue: '', description: 'Key for the start tag group to remove. Ref: grupo1start, grupo2start')
    //     string(name: 'REMOVE_VM_GROUP_STOP_TAG_KEY', defaultValue: '', description: 'Key for the stop tag group to remove. Ref: grupo1stop, grupo2stop')
    // }

    // environment {
    //     AZURE_CREDENTIALS = credentials(AZURE_CREDENTIALS_ID)
    // }

    // stages {
    //     stage('Login to Azure') {
    //         steps {
    //             script {
    //                 withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
    //                     sh '''
    //                     az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
    //                     az account set -s $AZURE_SUBSCRIPTION_ID
    //                     '''
    //                 }
    //             }
    //         }
    //     }
    
    stages {
        stage('Login to Azure') {
            steps {
                script {
                    sh '''
                    az login --service-principal -u 9e8b8c0e-0f92-4325-b421-2028bf37b447 -p 8ep8Q~CE1cbNQf~N3ZA3CbqAO9wxvMgj4AkUxc5c -t 9dbc76ea-fb25-4b07-8f07-5dc315999b76
                    az account set -s 0bea0a37-89cb-43fb-976f-0d8a3d8b1e4b
                    '''
                }
            }
        }
        
        // stage('Backup VM') {
        //     steps {
        //         script {
        //             def vmList = params.VM_LIST.split(',')
        //             vmList.each { vm ->
        //                 sh "az backup protection backup-now --resource-group rg_occidente_temp --vault-name vaultoccirpa --container-name IaasVMContainer;iaasvmcontainerv2;rg_occidente_temp;${vm} --item-name ${vm} --backup-management-type AzureIaasVM --workload-type VM"
        //             }
        //         }
        //     }
        // }

        stage('Backup VM') {
            steps {
                script {
                    sh 'az backup protection backup-now --resource-group rg_occidente_temp --vault-name vaultoccirpa --container-name "IaasVMContainer;iaasvmcontainerv2;rg_occidente_temp;vm1" --item-name vm1 --backup-management-type AzureIaasVM --workload-type VM'
                }
            }
        }

        stage('Backup FileShare') {
            steps {
                script {
                    sh 'az backup protection backup-now --resource-group rg_occidente_temp --vault-name vaultoccirpa --container-name "StorageContainer;Storage;rg_occidente_temp;rgoccidentetemp92cd" --item-name fileshareone --backup-management-type AzureStorage --workload-type AzureFileShare'
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
