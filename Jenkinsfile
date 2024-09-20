pipeline {
    agent any

    options {
        timeout(time: 30, unit: 'MINUTES')
    }

    parameters {
        string(name: 'ACTION', defaultValue: 'backup', description: 'Action to perform: backup or restore')
        string(name: 'TARGET', defaultValue: 'vm1', description: 'Target to perform action on: vm1, vm2, fileshare, sql1, or sql2')
    }

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
                        def jobId = null
                        if (params.TARGET == 'vm1') {
                            // Backup for vm1
                            def backupOutput = sh(script: "az backup protection backup-now --resource-group rg_occidente_temp --vault-name vaultoccirpa --container-name 'IaasVMContainer;iaasvmcontainerv2;rg_occidente_temp;vm1' --item-name vm1 --backup-management-type AzureIaasVM --workload-type VM", returnStdout: true).trim()
                            jobId = parseJobId(backupOutput)
                        } else if (params.TARGET == 'vm2') {
                            // Backup for vm2
                            def backupOutput = sh(script: "az backup protection backup-now --resource-group rg_occidente_temp --vault-name vaultoccirpa --container-name 'IaasVMContainer;iaasvmcontainerv2;rg_occidente_temp;vm2' --item-name vm2 --backup-management-type AzureIaasVM --workload-type VM", returnStdout: true).trim()
                            jobId = parseJobId(backupOutput)
                        } else if (params.TARGET == 'fileshare') {
                            // Backup for file share
                            def backupOutput = sh(script: "az backup protection backup-now --resource-group rg_occidente_temp --vault-name vaultoccirpa --container-name 'StorageContainer;Storage;rg_occidente_temp;rgoccidentetemp92cd' --item-name fileshareone --backup-management-type AzureStorage --workload-type AzureFileShare", returnStdout: true).trim()
                            jobId = parseJobId(backupOutput)
                        } else if (params.TARGET == 'sql1') {
                            // Backup for Azure SQL1
                            sh "az sql db export --admin-password ${params.SQL1_ADMIN_PASSWORD} --admin-user ${params.SQL1_ADMIN_USER} --authentication-type Sql --name ${params.SQL1_DB_NAME} --resource-group ${params.SQL1_RESOURCE_GROUP} --server ${params.SQL1_SERVER_NAME} --storage-key ${params.SQL1_STORAGE_KEY} --storage-key-type ${params.SQL1_STORAGE_KEY_TYPE} --storage-uri ${params.SQL1_STORAGE_URI}"
                        } else if (params.TARGET == 'sql2') {
                            // Backup for Azure SQL2
                            sh "az sql db export --admin-password ${params.SQL2_ADMIN_PASSWORD} --admin-user ${params.SQL2_ADMIN_USER} --authentication-type Sql --name ${params.SQL2_DB_NAME} --resource-group ${params.SQL2_RESOURCE_GROUP} --server ${params.SQL2_SERVER_NAME} --storage-key ${params.SQL2_STORAGE_KEY} --storage-key-type ${params.SQL2_STORAGE_KEY_TYPE} --storage-uri ${params.SQL2_STORAGE_URI}"
                        } else {
                            error "Invalid TARGET parameter: ${params.TARGET}. Must be 'vm1', 'vm2', 'fileshare', 'sql1', or 'sql2'."
                        }

                        if (jobId) {
                            waitForJobCompletion(jobId)
                        }
                    } else if (params.ACTION == 'restore') {
                        def jobId = null
                        if (params.TARGET == 'vm1') {
                            // Restore for vm1
                            def vmRecoveryPointsVm1 = sh(script: "az backup recoverypoint list --resource-group rg_occidente_temp --vault-name vaultoccirpa --container-name 'IaasVMContainer;iaasvmcontainerv2;rg_occidente_temp;vm1' --item-name vm1 --backup-management-type AzureIaasVM --workload-type VM", returnStdout: true).trim()
                            def vmRecoveryPointNameVm1 = parseRecoveryPointName(vmRecoveryPointsVm1)
                            if (vmRecoveryPointNameVm1) {
                                def restoreOutput = sh(script: "az backup restore restore-disks --resource-group rg_occidente_temp --vault-name vaultoccirpa --container-name 'IaasVMContainer;iaasvmcontainerv2;rg_occidente_temp;vm1' --item-name vm1 --rp-name ${vmRecoveryPointNameVm1} --storage-account rgoccidentetemp92cd --restore-mode OriginalLocation", returnStdout: true).trim()
                                jobId = parseJobId(restoreOutput)
                            } else {
                                error "No recovery points found for vm1."
                            }
                        } else if (params.TARGET == 'vm2') {
                            // Restore for vm2
                            def vmRecoveryPointsVm2 = sh(script: 'az backup recoverypoint list --resource-group rg_occidente_temp --vault-name vaultoccirpa --container-name "IaasVMContainer;iaasvmcontainerv2;rg_occidente_temp;vm2" --item-name vm2 --backup-management-type AzureIaasVM --workload-type VM', returnStdout: true).trim()
                            def vmRecoveryPointNameVm2 = parseRecoveryPointName(vmRecoveryPointsVm2)
                            if (vmRecoveryPointNameVm2) {
                                def restoreOutput = sh(script: "az backup restore restore-disks --resource-group rg_occidente_temp --vault-name vaultoccirpa --container-name 'IaasVMContainer;iaasvmcontainerv2;rg_occidente_temp;vm2' --item-name vm2 --rp-name ${vmRecoveryPointNameVm2} --storage-account rgoccidentetemp92cd --restore-mode OriginalLocation", returnStdout: true).trim()
                                jobId = parseJobId(restoreOutput)
                            } else {
                                error "No recovery points found for vm2."
                            }
                        } else if (params.TARGET == 'fileshare') {
                            // Restore for file share
                            def fileShareRecoveryPoints = sh(script: 'az backup recoverypoint list --resource-group rg_occidente_temp --vault-name vaultoccirpa --container-name "StorageContainer;Storage;rg_occidente_temp;rgoccidentetemp92cd" --item-name fileshareone --backup-management-type AzureStorage --workload-type AzureFileShare', returnStdout: true).trim()
                            def fileShareRecoveryPointName = parseRecoveryPointName(fileShareRecoveryPoints)
                            if (fileShareRecoveryPointName) {
                                def restoreOutput = sh(script: "az backup restore restore-azurefiles --resource-group rg_occidente_temp --vault-name vaultoccirpa --container-name 'StorageContainer;Storage;rg_occidente_temp;rgoccidentetemp92cd' --item-name fileshareone --rp-name ${fileShareRecoveryPointName} --resolve-conflict Overwrite --restore-mode OriginalLocation", returnStdout: true).trim()
                                jobId = parseJobId(restoreOutput)
                            } else {
                                error "No recovery points found for fileshare."
                            }
                        } else if (params.TARGET == 'sql1') {
                            // Restore for Azure SQL1
                            def restoreOutput = sh(script: "az sql db import --admin-password ${params.SQL1_ADMIN_PASSWORD} --admin-user ${params.SQL1_ADMIN_USER} --authentication-type Sql --name ${params.SQL1_DB_NAME} --resource-group ${params.SQL1_RESOURCE_GROUP} --server ${params.SQL1_SERVER_NAME} --storage-key ${params.SQL1_STORAGE_KEY} --storage-key-type ${params.SQL1_STORAGE_KEY_TYPE} --storage-uri ${params.SQL1_STORAGE_URI}", returnStdout: true).trim()
                            jobId = parseJobId(restoreOutput)
                        } else if (params.TARGET == 'sql2') {
                            // Restore for Azure SQL2
                            def restoreOutput = sh(script: "az sql db import --admin-password ${params.SQL2_ADMIN_PASSWORD} --admin-user ${params.SQL2_ADMIN_USER} --authentication-type Sql --name ${params.SQL2_DB_NAME} --resource-group ${params.SQL2_RESOURCE_GROUP} --server ${params.SQL2_SERVER_NAME} --storage-key ${params.SQL2_STORAGE_KEY} --storage-key-type ${params.SQL2_STORAGE_KEY_TYPE} --storage-uri ${params.SQL2_STORAGE_URI}", returnStdout: true).trim()
                            jobId = parseJobId(restoreOutput)
                        } else {
                            error "Invalid TARGET parameter: ${params.TARGET}. Must be 'vm1', 'vm2', 'fileshare', 'sql1', or 'sql2'."
                        }

                        if (jobId) {
                            waitForJobCompletion(jobId)
                            // Espera adicional para asegurar que los discos estén desasociados
                            sleep(time: 60, unit: 'SECONDS')
                            if (params.TARGET == 'vm1' || params.TARGET == 'vm2') {
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
        if (!managedBy || managedBy == 'none' || managedBy == 'null') {
            sh "az disk delete --name ${diskName} --resource-group rg_occidente_temp --yes"
            echo "Deleted managed disk: ${diskName}"
        } else {
            echo "Disk ${diskName} is still attached to a VM."
        }
    }
}
