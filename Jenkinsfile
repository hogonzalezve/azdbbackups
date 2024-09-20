pipeline {
    agent any

    options {
        timeout(time: 30, unit: 'MINUTES')
    }

    parameters {
        string(name: 'VM_NAME', defaultValue: 'vm1', description: 'Name of the VM to delete managed disks for')
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

        stage('Delete Managed Disks') {
            steps {
                script {
                    deleteManagedDisk(params.VM_NAME)
                }
            }
        }
    }
}

def deleteManagedDisk(vmName) {
    def diskList = sh(script: "az disk list --query \"[?contains(name, 'vm2')].{name:name, managedBy:managedBy}\" -o tsv", returnStdout: true).trim()
    def disks = diskList.split('\n')
    for (disk in disks) {
        def diskInfo = disk.split('\t')
        def diskName = diskInfo[0]
        def managedBy = diskInfo[1]
        if (!managedBy || managedBy == 'None') {
            sh "az disk delete --name ${diskName} --resource-group rg_occidente_temp --yes"
            echo "Deleted managed disk: ${diskName}"
        } else {
            echo "Disk ${diskName} is still attached to a VM."
        }
    }
}
