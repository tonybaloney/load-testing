# GitHub Action for Azure Load Testing

[GitHub Actions](https://help.github.com/en/articles/about-github-actions) gives you the flexibility to build an automated software development lifecycle workflow. You can automate your workflows to run load tests on Azure. [Azure Load Testing](https://docs.microsoft.com/azure/load-testing) is a fully managed load testing service that enables you to generate high-scale load. The service will simulate traffic for your applications, regardless of where they're hosted. Developers, testers, and quality assurance (QA) engineers can use it to optimize application performance, scalability, or capacity.

Get started today with a [free Azure account](https://azure.com/free/open-source)!

The definition of this GitHub Action is in [action.yml](/action.yml).

## Dependencies on other GitHub Actions
* [Azure Login](https://github.com/Azure/login) Login with your Azure Credentials for Authentication. Once login is done, the next set of Azure Actions in the workflow can re-use the same session within the job.

## Azure Service Principal for RBAC
For using any credentials like Azure Service Principal in your workflow, add them as [secrets](https://help.github.com/en/articles/virtual-enivronments-for-github-actions#creating-and-using-secrets-encrypted-variables) in the GitHub Repository and then refer them in the workflow.
1. Download Azure CLI from [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest), run `az login` to login with your Azure Credentials.
1. Run Azure CLI command to create an [Azure Service Principal for RBAC](https://docs.microsoft.com/en-us/azure/role-based-access-control/overview):
    ```bash

        az ad sp create-for-rbac --name "my-load-test-cicd" --role contributor \
                             --scopes /subscriptions/<subscription-id> \
                             --sdk-auth

        # Replace {subscription-id}, {resource-group} with the subscription, resource group details of the Azure Load Testing resource
        # The command should output a JSON object similar to this:

      {
        "clientId": "<GUID>",
        "clientSecret": "<GUID>",
        "subscriptionId": "<GUID>",
        "tenantId": "<GUID>",
        (...)
      }
    ```
1. Paste the json response from above Azure CLI to your GitHub Repository > Settings > Secrets > Add a new secret > **AZURE_CREDENTIALS**

1. To authorize the service principal to access the Azure Load Testing service, assign the Load Test Contributor role to the service principal.
    
    First, retrieve the service principal object ID by running this Azure CLI command:
    ```azurecli
    az ad sp list --filter "displayname eq 'my-load-test-cicd'" -o table
    ```
    
    Next, assign the **Load Test Contributor** role to the service principal. Replace the placeholder text *`<sp-object-id>`* with the **ObjectId** value from the previous Azure CLI command. Also, replace the *`<subscription-name-or-id>`* with your Azure subscription ID.

    ```azurecli
    az role assignment create --assignee "<sp-object-id>" \
        --role "Load Test Contributor" \
        --subscription "<subscription-name-or-id>"
    ```

1. Now in the workflow file in your branch: `.github/workflows/workflow.yml` replace the secret in Azure login action with your secret (Refer to the example below)

    ```yaml
    - uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    ```
    
## Azure Load Testing Action

This section describes the Azure Load Testing GitHub action. You can use this action by referencing `azure/load-testing@v1` action in your workflow. The action runs on Windows, Linux, and Mac runners.

You can use the following parameters to configure the GitHub action.

|Parameter  |Description  |
|---------|---------|
|`loadTestConfigFile`    | **Required** Path to the load test YAML configuration file. The path is fully qualified or relative to the default working directory.        |
|`resourceGroup`     |  **Required** Name of the resource group that contains the Azure Load Testing resource.       |
|`loadTestResource`     |   **Required** Name of an existing Azure Load Testing resource.      |
|`loadTestRunName` |  **Optional** Name of the load test run. If not specified, the default value is the date and time. |
|`loadTestRunDescription` | **Optional** Description of the load test run.  |
|`secrets`   |   Array of JSON objects that consist of the **name** and **value** for each secret that is required by your Apache JMeter script. The name should match the secret name used in the Apache JMeter test script. |
|`env`     |   Array of JSON objects that consist of the **name** and **value** for each environment variable that is required by your Apache JMeter script. The name should match the variable name used in the Apache JMeter test script. |
|`

The following YAML code snippet describes how to use the action in a GitHub Actions workflow.

```yaml
- name: 'Azure Load Testing'
  uses: azure/load-testing@v1
  with:
    loadTestConfigFile: '< YAML File path>'
    loadTestResource: '<name of the load test resource>'
    resourceGroup: '<name of the resource group of your load test resource>' 
    secrets: |
      [
          {
          "name": "<Name of the secret>",
          "value": "${{ secrets.MY_SECRET1 }}",
          },
          {
          "name": "<Name of the secret>",
          "value": "${{ secrets.MY_SECRET2 }}",
          }
      ]
    env: |
      [
          {
          "name": "<Name of the variable>",
          "value": "<Value of the variable>",
          },
          {
          "name": "<Name of the variable>",
          "value": "<Value of the variable>",
          }
      ]
```

The results files are exported to the folder “loadTest\results.zip”

### Sample workflow to run a load test using Azure Load testing service

```yaml
# File: .github/workflows/workflow.yml

on: push

jobs:
  # This workflow contains a single job called "loadtest"

  loadtest:
    name: Load Test
    runs-on: ubuntu-latest
    steps:
      - name: Checkout GitHub Actions 
        uses: actions/checkout@v2
          
      - name: Login to Azure
        uses: azure/login@v1
        continue-on-error: false
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
        
      - name: 'Azure Load Testing'
        uses: azure/load-testing@v1
        with:
          loadTestConfigFile: 'SampleApp.yaml'
          loadTestResource: 'loadTestResourceName'
          resourceGroup: 'loadTestResourceGroup'
```

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft 
trademarks or logos is subject to and must follow 
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.
