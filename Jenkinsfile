#!/usr/bin/env groovy
/* Only keep the 10 most recent builds. */

def projectProperties = [
    buildDiscarder(logRotator(numToKeepStr: '10')),
]

properties(projectProperties)

/* Librería manejo de variables */
import groovy.transform.Field

/* Creación Variables Globales */
@Field def VARIABLE_NAME = ''

pipeline {
    agent {
        label 'jenkins-slave-azcli'
    }

    options {
        timeout(time: 120, unit: 'MINUTES')
    }

    parameters {
        string(name: 'ACTION', defaultValue: '', description: 'Action to perform. Supported values (backup or restore)')
        string(name: 'TARGET', defaultValue: '', description: 'Target to perform action. Supported values (vm_rpavmsvcr001qa, fs_rpastfilesrepositoryqa, sql_rpa-sqldatabase-cr-qa, sql_rpa-sqldatabase-robots-qa)')
        string(name: 'DEPLOY_AZURE_SUBSCRIPTION_ID', defaultValue: '6719034d-d6a3-4665-8b43-251bf66a9d91', description: 'Subscription ID associated with backup resources')
    }

    stages {
        stage('Login to Azure') {
            steps {
                script {
                    withCredentials([
                        azureServicePrincipal(
                            credentialsId: 'CREDENTIAL_SERVICE_PRINCIPAL',
                            clientIdVariable: 'AZURE_CLIENT_ID',
                            clientSecretVariable: 'AZURE_CLIENT_SECRET',
                            tenantIdVariable: 'AZURE_TENANT_ID'
                        )
                    ]) 
                    {
                        sh """
                        az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
                        az account set -s ${params.DEPLOY_AZURE_SUBSCRIPTION_ID}
                        """
                    }
                }
            }
        }

        stage('Perform Action') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'CREDENTIAL_SQL_DATABASE',
                            usernameVariable: 'AZURE_SQL_USER',
                            passwordVariable: 'AZURE_SQL_PASSWORD'
                        ),
                        string(
                            credentialsId: 'CREDENTIAL_STORAGE_KEY',
                            variable: 'AZURE_STORAGE_KEY'
                        )
                    ])
                    {
                        if (params.ACTION == 'backup') {
                            def resourceGroup = 'rpa-rg-qa'
                            def recoveryVault = 'rpa-vm-recovery-vault-qa'
                            def iaasContainer = 'IaasVMContainer;iaasvmcontainerv2;rpa-rg-qa;rpavmsvcr001qa'
                            def storageContainer = 'StorageContainer;storage;rpa-rg-qa;rpastfilesrepositoryqa'
                            def iaasItem = 'rpavmsvcr001qa'
                            def storageItem = 'bots-repository'
                            def containerCr = 'backupscrqa'
                            def containerRb = 'backupsrbqa'
                            def nameStorage = 'rpastagingqa'
                            def dbCr = 'rpa-sqldatabase-cr-qa'
                            def dbRb = 'rpa-sqldatabase-robots-qa'
                            def srvCr = 'rpa-sqlserver-cr-qa'
                            def srvRb = 'rpa-sqlserver-robots360-qa'
                            def jobId = null
                            if (params.TARGET == 'vm_rpavmsvcr001qa') {
                                // Backup for rpavmsvcr001qa
                                def backupOutput = sh(script: "az backup protection backup-now --resource-group '${resourceGroup}' --vault-name '${recoveryVault}' --container-name '${iaasContainer}' --item-name '${iaasItem}' --backup-management-type AzureIaasVM --workload-type VM", returnStdout: true).trim()
                                jobId = parseJobId(backupOutput)
                            } else if (params.TARGET == 'vm_example') {
                                // Backup for example
                                def backupOutput = sh(script: "az backup protection backup-now --resource-group '${resourceGroup}' --vault-name '${recoveryVault}' --container-name '${iaasContainer}' --item-name '${iaasItem}' --backup-management-type AzureIaasVM --workload-type VM", returnStdout: true).trim()
                                jobId = parseJobId(backupOutput)
                            } else if (params.TARGET == 'fs_rpastfilesrepositoryqa') {
                                // Backup for rpastfilesrepositoryqa
                                def backupOutput = sh(script: "az backup protection backup-now --resource-group '${resourceGroup}' --vault-name '${recoveryVault}' --container-name '${storageContainer}' --item-name '${storageItem}' --backup-management-type AzureStorage --workload-type AzureFileShare", returnStdout: true).trim()
                                jobId = parseJobId(backupOutput)
                            } else if (params.TARGET == 'sql_rpa-sqldatabase-cr-qa') {
                                // Backup for Azure rpa-sqldatabase-cr-qa
                                def date = sh(script: "date +%Y%m%d-%H%M%S", returnStdout: true).trim()
                                def backupFileName = "backup-${dbCr}-${date}.bacpac"
                                def storageUri = "https://${nameStorage}.blob.core.windows.net//${containerCr}/${backupFileName}"
                                sh """
                                az sql db export --admin-user $AZURE_SQL_USER --admin-password $AZURE_SQL_PASSWORD --auth-type SQL --name '${dbCr}' --resource-group '${resourceGroup}' --server '${srvCr}' --storage-key $AZURE_STORAGE_KEY --storage-key-type StorageAccessKey --storage-uri '${storageUri}'
                                """
                            } else if (params.TARGET == 'sql_rpa-sqldatabase-robots-qa') {
                                // Backup for Azure rpa-sqldatabase-robots-qa
                                def date = sh(script: "date +%Y%m%d-%H%M%S", returnStdout: true).trim()
                                def backupFileName = "backup-${dbRb}-${date}.bacpac"
                                def storageUri = "https://${nameStorage}.blob.core.windows.net/${containerRb}/${backupFileName}"
                                sh """
                                az sql db export --admin-user $AZURE_SQL_USER --admin-password $AZURE_SQL_PASSWORD --auth-type SQL --name '${dbRb}' --resource-group '${resourceGroup}' --server '${srvRb}' --storage-key $AZURE_STORAGE_KEY --storage-key-type StorageAccessKey --storage-uri '${storageUri}'
                                """
                            } else {
                                error "Invalid TARGET parameter: ${params.TARGET}. Must be 'vm_rpavmsvcr001qa', 'fs_rpastfilesrepositoryqa', 'sql_rpa-sqldatabase-cr-qa' or 'sql_rpa-sqldatabase-robots-qa'."
                            }

                            if (jobId) {
                                waitForJobCompletion(jobId)
                            }
                        } else if (params.ACTION == 'restore') {
                            def resourceGroup = 'rpa-rg-qa'
                            def recoveryVault = 'rpa-vm-recovery-vault-qa'
                            def iaasContainer = 'IaasVMContainer;iaasvmcontainerv2;rpa-rg-qa;rpavmsvcr001qa'
                            def storageContainer = 'StorageContainer;storage;rpa-rg-qa;rpastfilesrepositoryqa'
                            def iaasItem = 'rpavmsvcr001qa'
                            def storageItem = 'bots-repository'
                            def containerCr = 'backupscrqa'
                            def containerRb = 'backupsrbqa'
                            def nameStorage = 'rpastagingqa'
                            def dbCr = 'rpa-sqldatabase-cr-qa'
                            def dbRb = 'rpa-sqldatabase-robots-qa'
                            def srvCr = 'rpa-sqlserver-cr-qa'
                            def srvRb = 'rpa-sqlserver-robots360-qa'
                            def jobId = null
                            if (params.TARGET == 'vm_rpavmsvcr001qa') {
                                // Restore for rpavmsvcr001qa
                                def vmRecoveryPointsVm1 = sh(script: "az backup recoverypoint list --resource-group '${resourceGroup}' --vault-name '${recoveryVault}' --container-name '${iaasContainer}' --item-name '${iaasItem}' --backup-management-type AzureIaasVM --workload-type VM", returnStdout: true).trim()
                                def vmRecoveryPointNameVm1 = parseRecoveryPointName(vmRecoveryPointsVm1)
                                if (vmRecoveryPointNameVm1) {
                                    def restoreOutput = sh(script: "az backup restore restore-disks --resource-group '${resourceGroup}' --vault-name '${recoveryVault}' --container-name '${iaasContainer}' --item-name '${iaasItem}' --rp-name ${vmRecoveryPointNameVm1} --storage-account ${nameStorage} --restore-mode OriginalLocation", returnStdout: true).trim()
                                    jobId = parseJobId(restoreOutput)
                                } else {
                                    error "No recovery points found for rpavmsvcr001qa."
                                }
                            } else if (params.TARGET == 'vm_example') {
                                // Restore for example
                                def vmRecoveryPointsVm2 = sh(script: "az backup recoverypoint list --resource-group '${resourceGroup}' --vault-name '${recoveryVault}' --container-name  '${iaasContainer}' --item-name '${iaasItem}' --backup-management-type AzureIaasVM --workload-type VM", returnStdout: true).trim()
                                def vmRecoveryPointNameVm2 = parseRecoveryPointName(vmRecoveryPointsVm2)
                                if (vmRecoveryPointNameVm2) {
                                    def restoreOutput = sh(script: "az backup restore restore-disks --resource-group '${resourceGroup}' --vault-name '${recoveryVault}' --container-name '${iaasContainer}' --item-name '${iaasItem}' --rp-name ${vmRecoveryPointNameVm2} --storage-account ${nameStorage} --restore-mode OriginalLocation", returnStdout: true).trim()
                                    jobId = parseJobId(restoreOutput)
                                } else {
                                    error "No recovery points found for example."
                                }
                            } else if (params.TARGET == 'fs_rpastfilesrepositoryqa') {
                                // Restore for file share
                                def fileShareRecoveryPoints = sh(script: "az backup recoverypoint list --resource-group ${resourceGroup} --vault-name ${recoveryVault} --container-name '${storageContainer}' --item-name '${storageItem}' --backup-management-type AzureStorage --workload-type AzureFileShare", returnStdout: true).trim()
                                def fileShareRecoveryPointName = parseRecoveryPointName(fileShareRecoveryPoints)
                                if (fileShareRecoveryPointName) {
                                    def restoreOutput = sh(script: "az backup restore restore-azurefiles --resource-group ${resourceGroup} --vault-name ${recoveryVault} --container-name '${storageContainer}' --item-name '${storageItem}' --rp-name ${fileShareRecoveryPointName} --resolve-conflict Overwrite --restore-mode OriginalLocation", returnStdout: true).trim()
                                    jobId = parseJobId(restoreOutput)
                                } else {
                                    error "No recovery points found for rpastfilesrepositoryqa."
                                }
                            } else if (params.TARGET == 'sql_rpa-sqldatabase-cr-qa') {
                                // Restore for Azure rpa-sqldatabase-cr-qa
                                def bacpacFile = getBacpacFile('${nameStorage}', '${containerCr}', '$AZURE_STORAGE_KEY')
                                def storageUri = "https://${nameStorage}.blob.core.windows.net/${containerCr}/${bacpacFile}"
                                echo "Using storage URI: ${storageUri}"
                                def restoreOutput = sh(script: "az sql db import --admin-user $AZURE_SQL_USER --admin-password $AZURE_SQL_PASSWORD --auth-type SQL --name ${dbCr} --resource-group ${resourceGroup} --server ${srvCr}--storage-key $AZURE_STORAGE_KEY --storage-key-type StorageAccessKey --storage-uri ${storageUri}", returnStdout: true).trim()

                                if (restoreOutput) {
                                    echo "Import completed successfully."

                                    // Delete the bacpac file after the import is complete
                                    deleteBacpacFile('${nameStorage}', '${containerCr}', '$AZURE_STORAGE_KEY')
                                }
                            } else if (params.TARGET == 'sql_rpa-sqldatabase-robots-qa') {
                                // Restore for Azure rpa-sqldatabase-robots-qa
                                def bacpacFile = getBacpacFile('${nameStorage}', '${containerRb}', '$AZURE_STORAGE_KEY')
                                def storageUri = "https://${nameStorage}.blob.core.windows.net/${containerRb}/${bacpacFile}"
                                echo "Using storage URI: ${storageUri}"
                                def restoreOutput = sh(script: "az sql db import --admin-user $AZURE_SQL_USER --admin-password $AZURE_SQL_PASSWORD --auth-type SQL --name ${dbRb} --resource-group ${resourceGroup} --server ${srvRb} --storage-key $AZURE_STORAGE_KEY --storage-key-type StorageAccessKey --storage-uri ${storageUri}", returnStdout: true).trim()

                                if (restoreOutput) {
                                    echo "Import completed successfully."

                                    // Delete the bacpac file after the import is complete
                                    deleteBacpacFile('${nameStorage}', '${containerRb}', '$AZURE_STORAGE_KEY')
                                }
                            } else {
                                error "Invalid TARGET parameter: ${params.TARGET}. Must be 'vm_rpavmsvcr001qa', 'fs_rpastfilesrepositoryqa', 'sql_rpa-sqldatabase-cr-qa' or 'sql_rpa-sqldatabase-robots-qa'."
                            }

                            if (jobId) {
                                waitForJobCompletion(jobId)
                                if (params.TARGET == 'vm_rpavmsvcr001qa' || params.TARGET == 'vm_example') {
                                    deleteManagedDisk(params.TARGET)
                                }
                            }
                        } else {
                            error "Invalid ACTION parameter: ${params.ACTION}. Must be 'backup' or 'restore'."
                        }
                    }
                }
            }
        }
    }
}

def parseRecoveryPointName(recoveryPointsJson) {
    def recoveryPoints = new groovy.json.JsonSlurper().parseText(recoveryPointsJson)
    return recoveryPoints[0]?.name ?: null
}

def parseJobId(output) {
    def json = new groovy.json.JsonSlurper().parseText(output)
    return json.id
}

def waitForJobCompletion(jobId) {
    sh(script: "az backup job wait --ids ${jobId}", returnStdout: true)
}

def deleteManagedDisk(vmName) {
    def diskList = sh(script: "az disk list --query \"[?contains(name, '${vmName}')].{name:name, managedBy:managedBy}\" -o tsv", returnStdout: true).trim()
    def disks = diskList.split('\n')
    for (disk in disks) {
        def diskInfo = disk.split('\t')
        def diskName = diskInfo[0]
        def managedBy = diskInfo[1]
        if (managedBy == 'None') {
            sh "az disk delete --name ${diskName} --resource-group ${resourceGroup} --yes"
            echo "Deleted managed disk: ${diskName}"
        } else {
            echo "Disk ${diskName} is attached to a VM."
        }
    }
}

def getBacpacFile(storageAccount, containerName, storageKey) {
    def listFilesCommand = """
    az storage blob list --account-name ${storageAccount} --container-name ${containerName} --account-key ${storageKey} --output tsv --query "[].{name:name}"
    """
    def filesList = sh(script: listFilesCommand, returnStdout: true).trim()
    echo "Files list: ${filesList}"
    def files = filesList.split('\n')

    def bacpacFile = null

    files.each { file ->
        if (file.endsWith('.bacpac')) {
            bacpacFile = file
            echo "Found bacpac file: ${bacpacFile}"
            return bacpacFile
        }
    }

    echo "Bacpac file selected: ${bacpacFile}"
    return bacpacFile
}

def deleteBacpacFile(storageAccount, containerName, storageKey) {
    def deleteFileCommand = """
    az storage blob delete-batch --account-name ${storageAccount} --source ${containerName}  --pattern "*.bacpac" --account-key ${storageKey}
    """
    sh(script: deleteFileCommand, returnStdout: true)
}
