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

    parameters {
        string(name: 'ACTION', defaultValue: 'backup', description: 'Action to perform: backup or restore')
        string(name: 'TARGET', defaultValue: 'vm1', description: 'Target to perform action on: vm1, vm2, or fileshare')
    }

    // environment {
    //     AZURE_CREDENTIALS = credentials(AZURE_CREDENTIALS_ID)
    // }

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

        stage('Perform Action') {
            steps {
                script {
                    if (params.ACTION == 'backup') {
                        if (params.TARGET == 'vm1') {
                            // Backup for vm1
                            sh 'az backup protection backup-now --resource-group rg_occidente_temp --vault-name vaultoccirpa --container-name "IaasVMContainer;iaasvmcontainerv2;rg_occidente_temp;vm1" --item-name vm1 --backup-management-type AzureIaasVM --workload-type VM'
                        } else if (params.TARGET == 'vm2') {
                            // Backup for vm2
                            sh 'az backup protection backup-now --resource-group rg_occidente_temp --vault-name vaultoccirpa --container-name "IaasVMContainer;iaasvmcontainerv2;rg_occidente_temp;vm2" --item-name vm2 --backup-management-type AzureIaasVM --workload-type VM'
                        } else if (params.TARGET == 'fileshare') {
                            // Backup for file share
                            sh 'az backup protection backup-now --resource-group rg_occidente_temp --vault-name vaultoccirpa --container-name "StorageContainer;Storage;rg_occidente_temp;rgoccidentetemp92cd" --item-name fileshareone --backup-management-type AzureStorage --workload-type AzureFileShare'
                        } else {
                            error "Invalid TARGET parameter: ${params.TARGET}. Must be 'vm1', 'vm2', or 'fileshare'."
                        }
                    } else if (params.ACTION == 'restore') {
                        if (params.TARGET == 'vm1') {
                            // Restore for vm1
                            def recoveryPointsVm1 = sh(script: 'az backup recoverypoint list --resource-group rg_occidente_temp --vault-name vaultoccirpa --container-name "IaasVMContainer;iaasvmcontainerv2;rg_occidente_temp;vm1" --item-name vm1 --backup-management-type AzureIaasVM --workload-type VM', returnStdout: true).trim()
                            def recoveryPointIdVm1 = parseRecoveryPointId(recoveryPointsVm1)
                            sh 'az backup recoverypoint restore --resource-group rg_occidente_temp --vault-name vaultoccirpa --container-name "IaasVMContainer;iaasvmcontainerv2;rg_occidente_temp;vm1" --item-name vm1 --backup-management-type AzureIaasVM --workload-type VM --recovery-point-id ${recoveryPointIdVm1}'
                        } else if (params.TARGET == 'vm2') {
                            // Restore for vm2
                            def recoveryPointsVm2 = sh(script: 'az backup recoverypoint list --resource-group rg_occidente_temp --vault-name vaultoccirpa --container-name "IaasVMContainer;iaasvmcontainerv2;rg_occidente_temp;vm2" --item-name vm2 --backup-management-type AzureIaasVM --workload-type VM', returnStdout: true).trim()
                            def recoveryPointIdVm2 = parseRecoveryPointId(recoveryPointsVm2)
                            sh 'az backup recoverypoint restore --resource-group rg_occidente_temp --vault-name vaultoccirpa --container-name "IaasVMContainer;iaasvmcontainerv2;rg_occidente_temp;vm2" --item-name vm2 --backup-management-type AzureIaasVM --workload-type VM --recovery-point-id ${recoveryPointIdVm2}'
                        } else if (params.TARGET == 'fileshare') {
                            // Restore for file share
                            def fileShareRecoveryPoints = sh(script: 'az backup recoverypoint list --resource-group rg_occidente_temp --vault-name vaultoccirpa --container-name "StorageContainer;Storage;rg_occidente_temp;rgoccidentetemp92cd" --item-name fileshareone --backup-management-type AzureStorage --workload-type AzureFileShare', returnStdout: true).trim()
                            def fileShareRecoveryPointName = parseRecoveryPointName(fileShareRecoveryPoints)
                            if (fileShareRecoveryPointName) {
                                sh "az backup restore restore-azurefiles --resource-group rg_occidente_temp --vault-name vaultoccirpa --container-name 'StorageContainer;Storage;rg_occidente_temp;rgoccidentetemp92cd' --item-name fileshareone --rp-name ${fileShareRecoveryPointName} --resolve-conflict Overwrite --restore-mode OriginalLocation"
                            } else {
                                error "No recovery points found for fileshare."
                            }
                        } else {
                            error "Invalid TARGET parameter: ${params.TARGET}. Must be 'vm1', 'vm2', or 'fileshare'."
                        }
                    } else {
                        error "Invalid ACTION parameter: ${params.ACTION}. Must be 'backup' or 'restore'."
                    }
                }
            }
        }
    }
}

def parseRecoveryPointId(recoveryPointsJson) {
    def recoveryPoints = new groovy.json.JsonSlurper().parseText(recoveryPointsJson)
    return recoveryPoints[0]?.id ?: error("No recovery points found")
}

def parseRecoveryPointName(recoveryPointsJson) {
    def recoveryPoints = new groovy.json.JsonSlurper().parseText(recoveryPointsJson)
    return recoveryPoints[0]?.name ?: null
}
