# Monitoring and automating Azure solutions

# Lab Answer Key: Deploying Configuration Management solutions to Azure

## Exercise 1: Deploy compute resources

#### Task 1: Open the Azure portal

1. On the Taskbar, click the **Microsoft Edge** icon.

1. In the open browser window, navigate to the **Azure Portal** (<https://portal.azure.com>).

1. When prompted, authenticate with the user account account provided by your instructor.

#### Task 2: Open Cloud Shell

1. At the top of the portal, click the **Cloud Shell** icon to open a new shell instance.

1. Refer to Exercice 1 in Lab 00 to create your Powershell environment.

1. In the title line of the Cloud Shell pane, select **Bash** and **Confirm**.

#### Task 3: Deploy a Linux VM

1. You will deploy the **linux-template.json** teplate file, conteining the following resources:

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "userName": {
                "type": "string",
                "defaultValue": "Student"
            },
            "password": {
                "type": "securestring"
            }
        },
        "variables": {
            "vmName": "[concat('lvm', uniqueString(resourceGroup().id))]",
            "nicName": "[concat('nic', uniqueString(resourceGroup().id))]",
            "publicIPAddressName": "[concat('pip', uniqueString(resourceGroup().id))]",
            "virtualNetworkName": "[concat('vnt', uniqueString(resourceGroup().id))]",
            "subnetName": "Linux",
            "imageReference": {
                "publisher": "suse",
                "offer": "opensuse-leap",
                "sku": "42.3",
                "version": "latest"
            }
        },
        "resources": [
            {
                "apiVersion": "2017-06-01",
                "type": "Microsoft.Network/publicIPAddresses",
                "name": "[variables('publicIPAddressName')]",
                "location": "[resourceGroup().location]",
                "properties": {
                    "publicIPAllocationMethod": "Dynamic"
                }
            },
            {
                "apiVersion": "2017-06-01",
                "type": "Microsoft.Network/virtualNetworks",
                "name": "[variables('virtualNetworkName')]",
                "location": "[resourceGroup().location]",
                "properties": {
                    "addressSpace": {
                        "addressPrefixes": [
                            "10.0.0.0/16"
                        ]
                    },
                    "subnets": [
                        {
                            "name": "[variables('subnetName')]",
                            "properties": {
                                "addressPrefix": "10.0.0.0/24"
                            }
                        }
                    ]
                }
            },
            {
                "apiVersion": "2017-10-01",
                "type": "Microsoft.Network/networkInterfaces",
                "name": "[variables('nicName')]",
                "location": "[resourceGroup().location]",
                "dependsOn": [
                    "[resourceId('Microsoft.Network/publicIPAddresses/', variables('publicIPAddressName'))]",
                    "[resourceId('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]"
                ],
                "properties": {
                    "ipConfigurations": [
                        {
                            "name": "ipconfig1",
                            "properties": {
                                "privateIPAllocationMethod": "Dynamic",
                                "publicIPAddress": {
                                    "id": "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPAddressName'))]"
                                },
                                "subnet": {
                                    "id": "[concat(resourceId('Microsoft.Network/virtualNetworks',variables('virtualNetworkName')), '/subnets/', variables('subnetName'))]"
                                }
                            }
                        }
                    ]
                }
            },
            {
                "apiVersion": "2017-03-30",
                "type": "Microsoft.Compute/virtualMachines",
                "name": "[variables('vmName')]",
                "location": "[resourceGroup().location]",
                "dependsOn": [
                    "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
                ],
                "properties": {
                    "hardwareProfile": {
                        "vmSize": "Standard_DS1_v2"
                    },
                    "osProfile": {
                        "computerName": "[variables('vmName')]",
                        "adminUsername": "[parameters('username')]",
                        "adminPassword": "[parameters('password')]"
                    },
                    "storageProfile": {
                        "imageReference": "[variables('imageReference')]",
                        "osDisk": {
                            "createOption": "FromImage"
                        }
                    },
                    "networkProfile": {
                        "networkInterfaces": [
                            {
                                "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
                            }
                        ]
                    }
                }
            }
        ]
    }
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a variable which value designates the name of the resource group that will contain the hub virtual network:

    ```sh
    RESOURCE_GROUP='StagiaireXXX-RG1'
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to deploy the Azure Resource Manager template with the specified parameters file:

    ```sh
    az group deployment create --resource-group $RESOURCE_GROUP --template-file ~/az-301/T2/linux-template.json --parameters password=Pa55w.rd1234
    ```

1. Do not wait for the deployment to complete before you proceed to the next task.

#### Task 4: Deploy an Azure Automation account

1. In the upper left corner of the Azure portal, click **Create a resource**.

1. At the top of the **New** blade, in the **Search the Marketplace** text box, type **Automation** and press **Enter**.

1. On the **Everything** blade, in the search results, click **Automation**.

1. On the **Automation** blade, click **Create**.

1. On the **Add Automation Account** blade, perform the following tasks:

    - In the **Name** text box, type **LinuxAutomation**.

    - Leave the **Subscription** drop-down list entry set to its default value.

    - In the **Resource group** section, select **StagiaireXXX-RG1**.

    - In the **Location** drop-down list, select the Azure region from the Resource group.

    - In the **Create Azure Run As account** section, ensure that **No** option is selected.

    - Click the **Create** button.

1. Wait for the provisioning to complete before you proceed to the next task.

> **Review**: In this exercise, you created a Linux VM using an Azure Resource Manager template and provisioned an Azure Automation account from the Azure portal.


## Exercise 2: Configure Azure Automation DSC

#### Task 1: Import Linux PowerShell DSC modules

1. In the hub menu of the Azure portal, click **Resource groups**.

1. On the **Resource groups** blade, click **StagiaireXXX-RG1**.

1. On the **StagiaireXXX-RG1** blade, click the newly created Azure Automation account.

1. On the **LinuxAutomation** blade, in the **SHARED RESOURCES** section on the left side of the blade, click **Modules gallery**.

1. On the **LinuxAutomation - Modules gallery** blade, perform the following tasks:

    - In the **Search** text box, type **nx** and press **Enter**.

    - In the search results, click the **nx** module.

1. On the **nx** blade, click the **Import** button at the top of the blade.

1. On the **Import** blade, click the **OK** button.

1. Wait for the import process to finish before you proceed to the next task. A status message on the **nx Module** blade will indicate that the module was successfully imported.

    > **Note**: This process should take about 2 minutes.

#### Task 2: Create Linux DSC Configuration

1. Navigate back to the **LinuxAutomation** blade.

1. Back on the **LinuxAutomation** blade, in the **CONFIGURATION MANAGEMENT** section, click **State configuration (DSC)**.

1. On the **LinuxAutomation - State configuration (DSC)** blade, click the **Configurations** tab.

1. On the **LinuxAutomation - State configuration (DSC)** blade, click the **+ Add** button at the top of the pane.

1. On the **Import** blade, perform the following tasks:

    - Next to the **Configuration file** field, click the blue button with a folder icon.

    - In the **Choose File to Upload** dialog box, navigate to the **\\allfiles\\AZ-301T02\\Module_03\\LabFiles\\Starter\\** folder.

    - Select the **lampserver.ps1** file.

    - Click the **Open** button to close the dialog and return to the **Import** blade.

    - In the **Name** text box, accept the default entry **lampserver**.

    - In the **Description** text box, type **LAMP Server configuration using PHP and MySQL**.

    - Click the **OK** button.

1. Back in the **DSC configurations** pane, click **Refresh** and then click the newly created **lampserver** configuration.

1. On the **lampserver Configuration** blade, click the **Compile** button at the top of the blade. In the confirmation dialog box, click **Yes** to proceed with compiling the configuration.

1. Wait for the compilation task to finish. To determine the status of the compilation task, review the **STATUS** column of the **Compilation jobs** section of the **lampserver Configuration** blade.

    > **Note**: You may need to close and re-open the blade to see the latest compilation status. This blade does not refresh automatically. Alternatively, you should open the job blade.

#### Task 3: Onboard Linux VM

1. Navigate back to the **LinuxAutomation - State Configuration (DSC)** blade.

1. Back on the **LinuxAutomation - State Configuration (DSC)** blade, click the **Nodes** link.

1. On the **LinuxAutomation - State configuration (DSC)** blade, click the **+ Add** button at the top of the pane.

1. On the **Virtual Machines** blade, click the entry representing the Linux virtual machine you deployed in the previous exercise.

1. On the virtual machine blade, click **+ Connect**.

1. On the **Registration** blade, perform the following tasks:

    - Leave the **Registration key** setting with its default value.

    - In the **Node configuration name** drop-down list, select the **lampserver.localhost** entry.

    - Leave all remaining settings with their default values.

    - Click the **OK** button.

1. Wait for the connection process to complete before you proceed to the next step.

1. Navigate back to the **LinuxAutomation - State Configuration (DSC)** blade.

1. On the **LinuxAutomation - State configuration (DSC)** blade, select in the **NODE** section the virtual machine you deployed in the previous exercise.

1. On the virtual machine blade, click **Assign node configuration**.

1. On the Assign Node Configuration blade, select the node configuration **lampserver.host** and click the **OK** button.

1. Back on the **LinuxAutomation - State Configuration (DSC)** blade, click the **Refresh** button.

1. In the list of DSC nodes, verify that the Linux virtual machine has the **Compliant** status.

> **Review**: In this exercise, you created a PowerShell DSC configuration and applied the configuration to a Linux virtual machine.


#### Clean up resources

1. Refer to **Exercice 2** of **Lab 00** to clean up your resources.
