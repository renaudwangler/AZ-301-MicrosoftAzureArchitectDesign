# Managing Security and Identity for Azure Solutions

# Lab Answer Key: Securing Secrets in Azure

## Exercise 1: Deploy Key Vault resources

#### Task 1: Open the Azure Portal

1. On the Taskbar, click the **Microsoft Edge** icon.

1. In the open browser window, navigate to the **Azure Portal** (<https://portal.azure.com>).

1. When prompted, authenticate with the user account account provided by your instructor.

#### Task 2: Deploy a key vault

1. In the upper left corner of the Azure portal, click **Create a resource**.

1. At the top of the **New** blade, in the **Search the Marketplace** text box, type **Key Vault** and press **Enter**.

1. On the **Everything** blade, in the search results, click **Key Vault**.

1. On the **Key Vault** blade, click the **Create** button.

1. On the **Create key vault** blade, perform the following tasks:

    - Leave the **Subscription** drop-down list entry set to its default value.

    - In the **Resource group** section, select **StagiaireXXX-RG1**.

    - In the **Key vault name** text box, type a globally unique value.

    - In the **Region** drop-down list, select the Azure region of the Resource group.

    - Leave all remaining settings with their default values.

    - Click the **Review + Create** and the **Create** button.

1. Wait for the provisioning to complete before you proceed to the next task.

#### Task 3: Add a secret to a key vault by using the Azure portal

1. In the hub menu in the Azure portal, click **Resource groups**.

1. On the **Resource groups** blade, click **StagiaireXXX-RG1**.

1. On the **StagiaireXXX-RG1** blade, click the entry representing the newly created key vault.

1. On the key vault blade, click **Secrets**.

1. On the key vault secrets blade, click the **Generate/Import** button at the top of the pane.

1. On the **Create a secret** blade, perform the following tasks:

    - In the **Upload options** drop-down list, ensure that the **Manual** entry is selected.

    - In the **Name** text-box, type **thirdPartyKey**.

    - In the **Value** text box, enter the value **56d95961e597ed0f04b76e58**.

    - Leave all remaining settings with their default values.

    - Click the **Create** button.

#### Task 4: Open Cloud Shell

1. At the top of the portal, click the **Cloud Shell** icon to open a new shell instance.

    > **Note**: The **Cloud Shell** icon is a symbol that is constructed of the combination of the *greater than* and *underscore* characters.

1. Refer to Exercice 1 in Lab 00 to create your Powershell environment.

1. In the title line of the Cloud Shell pane, select **Bash** and **Confirm**.

#### Task 5: Add a secret to a key vault using the CLI

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a variable which value designates the name of the resource group that contains the Azure key vault you deployed earlier in this exercise:

    ```sh
    RESOURCE_GROUP='StagiaireXXX-RG1'
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to retrieve the name of the Azure key vault you created earlier in this exercise:

    ```sh
    KEY_VAULT_NAME=$(az keyvault list --resource-group $RESOURCE_GROUP --query "[0].name" --output tsv)
    ```

1. At the **Cloud Shell** command prompt, type in the following command, and press **Enter** to list secrets in the key vault:

    ```sh
    az keyvault secret list --vault-name $KEY_VAULT_NAME --output json
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to display the value of the **thirdPartyKey** secret:

    ```sh
    az keyvault secret show --vault-name $KEY_VAULT_NAME --name thirdPartyKey --query value --output tsv
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to add a new secret to your key vault:

    ```sh
    az keyvault secret set --vault-name $KEY_VAULT_NAME --name firstPartyKey --value 56f8a55119845511c81de488
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to list secrets in the key vault:

    ```sh
    az keyvault secret list --vault-name $KEY_VAULT_NAME --query "[*].{Id:id,Created:attributes.created}" --out table
    ```

1. Close the **Cloud Shell** pane.

#### Task 6: Add secrets to a key vault by using Azure Resource Manager templates

1. In the upper left corner of the Azure portal, click **Create a resource**.

1. At the top of the **New** blade, in the **Search the Marketplace** text box, type **Template Deployment** and press **Enter**.

1. On the **Everything** blade, in the search results, click **Template Deployment**.

1. On the **Template deployment** blade, click the **Create** button.

1. On the **Custom deployment** blade, click the **Build your own template in the editor** link.

1. On the **Edit template** blade, click **Load file**.

1. In the **Choose File to Upload** dialog box, navigate to the **\\allfiles\\AZ-301T01\\Module_01\\LabFiles\\Starter\\** folder, select the **secret-template.json** file, and click **Open**. This will load the following content into the template editor pane:

    ```json
    {
        "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "vaultName": {
                "type": "string"
            }
        },
        "variables": {
            "secretName": "vmPassword"
        },
        "resources": [
            {
                "apiVersion": "2016-10-01",
                "type": "Microsoft.KeyVault/vaults/secrets",
                "name": "[concat(parameters('vaultName'), '/', variables('secretName'))]",
                "properties": {
                    "contentType": "text/plain",
                    "value": "StudentPa$$w.rd"
                }
            }
        ]
    }
    ```

1. Click the **Save** button to persist the template.

1. Back on the **Custom deployment** blade, perform the following tasks:

    - Leave the **Subscription** drop-down list entry set to its default value.

    - In the **Resource group** section, select the **Use existing** option and then, in the drop-down list, select **StagiaireXXX-RG1**.

    - In the **Vault Name** text box, type the name of the key vault you created earlier in this exercise.

    - In the **Terms and Conditions** section, select the **I agree to the terms and conditions stated above** checkbox.

    - Click the **Purchase** button.

1. Do not wait for the deployment to complete but proceed to the next step.

1. In the upper left corner of the Azure portal, click **Create a resource**.

1. At the top of the **New** blade, in the **Search the Marketplace** text box, type **Template Deployment** and press **Enter**.

1. On the **Everything** blade, in the search results, click **Template Deployment**.

1. On the **Template deployment** blade, click the **Create** button.

1. On the **Custom deployment** blade, click the **Build your own template in the editor** link.

1. On the **Edit template** blade, click **Load file**.

1. In the **Choose File to Upload** dialog box, navigate to the **\\allfiles\\AZ-301T01\\Module_01\\LabFiles\\Starter\\** folder, select the **storage-template.json** file, and click **Open**. This will load the following content into the template editor pane:

    ```json
    {
        "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "vaultName": {
                "type": "string"
            }
        },
        "variables": {
            "secretName": "storageConnectionString",
            "storageName": "[concat('stor', uniqueString(resourceGroup().id))]"
        },
        "resources": [
            {
                "apiVersion": "2017-10-01",
                "type": "Microsoft.Storage/storageAccounts",
                "name": "[variables('storageName')]",
                "location": "[resourceGroup().location]",
                "kind": "Storage",
                "sku": {
                    "name": "Standard_LRS"
                },
                "properties": {
                }
            },
            {
                "apiVersion": "2016-10-01",
                "type": "Microsoft.KeyVault/vaults/secrets",
                "name": "[concat(parameters('vaultName'), '/', variables('secretName'))]",
                "dependsOn": [
                    "[resourceId('Microsoft.Storage/storageAccounts', variables('storageName'))]"
                ],
                "properties": {
                    "contentType": "text/plain",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageName'), ';', 'AccountKey=', listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageName')), providers('Microsoft.Storage', 'storageAccounts').apiVersions[0]).keys[0].value, ';')]"
                }
            }
        ]
    }
    ```

1. Click the **Save** button to persist the template.

1. Back on the **Custom deployment** blade, perform the following tasks:

    - Leave the **Subscription** drop-down list entry set to its default value.

    - In the **Resource group** section, select the **Use existing** option and then, in the drop-down list, select **StagiaireXXX-RG1**.

    - In the **Vault Name** field, type the name of the key vault you created earlier in this exercise.

    - In the **Terms and Conditions** section, select the **I agree to the terms and conditions stated above** checkbox.

    - Click the **Purchase** button.

1. Wait for the deployment to complete before you proceed to the next task.

#### Task 7: View key vault secrets

1. In the hub menu of the Azure portal, click **Resource groups**.

1. On the **Resource groups** blade, click **StagiaireXXX-RG1**.

1. On the **StagiaireXXX-RG1** blade, click the entry representing the key vault you created earlier in this exercise.

1. On the key vault blade, click **Secrets**.

1. On the key vault secrets blade, review the list of secrets created during this lab.

1. Click the entry representing the **vmPassword** secret.

1. On the **vmPassword** blade, click the entry representing the current version of the secret.

1. On the Secret Version blade, click the **Show secret value** button.

1. Verify that the value of the secret matches the one included in the template you deployed in the previous task.

> **Review**: In this exercise, you created a **Key Vault** instance and used several different methods to add secrets to the key vault.


## Exercise 2: Deploy Azure VM using Key Vault secret

#### Task 1: Retrieve the value of the key vault Resource Id parameter

1. At the top of the portal, click the **Cloud Shell** icon to open a new Clould Shell instance.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a variable which value designates the name of the resource group that will contain the hub virtual network:

    ```sh
    RESOURCE_GROUP='StagiaireXXX-RG1'
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to retrieve the resource id of the Azure key vault you created earlier in this exercise:

    ```sh
    KEY_VAULT_ID=$(az keyvault list --resource-group $RESOURCE_GROUP --query "[0].id" --output tsv)
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a variable which value designates the name of the Azure key vault resource id and which takes into account any special character the resource id might include:

    ```sh
    KEY_VAULT_ID_REGEX="$(echo $KEY_VAULT_ID | sed -e 's/\\/\\\\/g; s/\//\\\//g; s/&/\\\&/g')"
    ```

#### Task 2: Prepare the Azure Resource Manager deployment template and parameters files

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to replace the placeholder for the **$KEY_VAULT_ID** parameter in the **vm-template.parameters.json** parameters file with the value of the **$KEY_VAULT_ID** variable:

    ```sh
    sed -i.bak1 's/"$KEY_VAULT_ID"/"'"$KEY_VAULT_ID_REGEX"'"/' ~/az-301/T1/vm-template.parameters.json
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to verify that the placeholder was successfully replaced in the parameters file:

    ```sh
    cat ~/az-301/T1/vm-template.parameters.json
    ```

#### Task 3: Configure a key vault for deployment of Azure Resource Manager templates

1. In the hub menu in the Azure portal, click **Resource groups**.

1. On the **Resource groups** blade, click **StagiaireXXX-RG1**.

1. On the **StagiaireXXX-RG1** blade, click the entry representing the key vault you created in the previous exercise.

1. On the key vault blade, click **Access policies**.

1. On the **Access policies** blade, under the **Enable access to:** area, select the **Azure Resource Manager for template deployment** checkbox.

1. Click the **Save** button at the top of the pane.

#### Task 4: Deploy a Linux VM with the password parameter set by using a key vault secret.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to deploy the Azure Resource Manager template with the specified parameters file:

    ```sh
    az group deployment create --resource-group $RESOURCE_GROUP --template-file ~/az-301/T1/vm-template.json --parameters @~/az-301/T1/vm-template.parameters.json
    ```

1. Wait for the deployment to complete before you proceed to the next task.

#### Task 5: Verify the outcome of the deployment

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a variable which value designates the name of the resource group that contains the newly deployed Azure VM:

    ```
    RESOURCE_GROUP='StagiaireXXX-RG1''
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to retrieve the name of the Azure key vault containing the secret that stores the value of the password of the local Administrator account:

    ```sh
    KEY_VAULT_NAME=$(az keyvault list --resource-group $RESOURCE_GROUP --query "[0].name" --output tsv)
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to retrieve the value of the secret:

    ```sh
    az keyvault secret show --vault-name $KEY_VAULT_NAME --name vmPassword --query value --output tsv
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to retrieve the public IP address of the Azure VM you deployed in the previous task:

    ```sh
    PUBLIC_IP=$(az network public-ip list --resource-group $RESOURCE_GROUP --query "[0].ipAddress" --output tsv)
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to connect to the Azure VM via SSH:

    ```sh
    ssh Student@$PUBLIC_IP
    ```

1. At the **Cloud Shell** command prompt, when prompted whether you want to continue connecting, type `yes` and press **Enter**.

1. At the **Cloud Shell** command prompt, when prompted for password, type the value of the secret you retrieved earlier in this task and press **Enter**.

1. Verify that you successfully authenticated.

1. At the **Cloud Shell** command prompt, type `exit` to log out from the Azure VM.

> **Review**: In this exercise, you deployed a Linux VM using a password stored as a key vault secret.

#### Clean up resources

1. Refer to **Exercice 2** of **Lab 00** to clean up your resources.
