---
name: "Azure Functions JavaScript HTTP Trigger using AZD"
description: This repository contains an Azure Functions HTTP trigger quickstart written in JavaScript and deployed to Azure Functions Flex Consumption using the Azure Developer CLI (AZD). This sample uses managed identity and a virtual network to insure it is secure by default.
page_type: sample
languages:
- nodejs
- JavaScript
products:
- azure
- azure-functions
- entra-id
urlFragment: functions-quickstart-javascript-azd
---

# Azure Functions JavaScript HTTP Trigger using AZD

This repository contains a Azure Functions HTTP trigger quickstart written in JavaScript and deployed to Azure using Azure Developer CLI (AZD). This sample uses managed identity and a virtual network to insure it is secure by default. 

## Getting Started

### Prerequisites

+ [Node.js 20](https://www.nodejs.org/)
+ [Azure Functions Core Tools](https://learn.microsoft.com/azure/azure-functions/functions-run-local#install-the-azure-functions-core-tools)
+ [Azure Developer CLI (AZD)](https://learn.microsoft.com/azure/developer/azure-developer-cli/install-azd)
+ [Visual Studio Code](https://code.visualstudio.com/)
  + Needed only when using Visual Studio Code to run and debug locally.
  + Also requires the [Azure Functions extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)
+ One of these HTTP test tools for sending HTTP POST requests to the URL endpoint:
  + [Visual Studio Code](https://code.visualstudio.com/download) with an [extension from Visual Studio Marketplace](https://marketplace.visualstudio.com/vscode)
  + [PowerShell Invoke-RestMethod](/powershell/module/microsoft.powershell.utility/invoke-restmethod)
  + [Microsoft Edge - Network Console tool](/microsoft-edge/devtools-guide-chromium/network-console/network-console-tool)
  + [Bruno](https://www.usebruno.com/)
  + [curl](https://curl.se/)
    
  > [!CAUTION]  
  > For scenarios where you have sensitive data, such as credentials, secrets, access tokens, API keys, and other similar information, make sure to use a tool that protects your data with the necessary security features, works offline or locally, doesn't sync your data to the cloud, and doesn't require that you sign in to an online account. This way, you reduce the risk around exposing sensitive data to the public.

### Get repo on your local machine
Run the following GIT command to clone this repository to your local machine.
```bash
git clone https://github.com/Azure-Samples/functions-quickstart-javascript-azd.git
```

## Run on your local environment

### Prepare your local environment
1) Navigate to the new _functions-quickstart-javascript-azd_ folder.
1) Create a file named `local.settings.json` and add the following:
    ```json
    {
      "IsEncrypted": false,
      "Values": {
        "AzureWebJobsStorage": "UseDevelopmentStorage=true",
        "FUNCTIONS_WORKER_RUNTIME": "node"
      }
    }
    ```

### Using Functions CLI
1) Open the _functions-quickstart-javascript-azd_ folder in a new terminal and run the following commands:

    ```bash
    npm install
    func start
    ```
  You should see the URL endpoints for the two HTTP triggered functions in the output.

2) Test the HTTP GET trigger by opening the <http://localhost:7071/api/httpGetFunction> link in your browser or HTTP test tool.

3) Test the HTTP POST trigger in a new terminal window using your HTTP test tool, such as this curl command:
   
    ```bash
    curl -i -X POST http://localhost:7071/api/httppostbodyfunction -H "Content-Type: text/json" --data-binary "@src/functions/testdata.json"
    ```

### Using Visual Studio Code
1) Open this folder in a new terminal
2) Open Visual Studio Code by running `code .` in the terminal
4) Add required files to the `.vscode` folder by opening the command palette using `Crtl+Shift+P` (or `Cmd+Shift+P` on Mac) and selecting *"Azure Functions: Initialize project for use with VS Code"*
5) Press Run/Debug (F5) to run in the debugger (select "Debug anyway" if prompted about local emulater not running) 
6) Use same approach above to test using an HTTP test extension in Visual Studio Code.

## Source Code

The key code that makes tthese functions work is in `./src/functions/httpGetFunction.cs` and `.src/functions/httpPostBodyFunction.cs`.  The function is identified as an Azure Function by use of the `@azure/functions` library. This code shows a HTTP GET triggered function.  

```javascript
const { app } = require('@azure/functions');

app.http('httpGetFunction', {
    methods: ['GET'],
    authLevel: 'function',
    handler: async (request, context) => {
        context.log(`Http function processed request for url "${request.url}"`);

        const name = request.query.get('name') || await request.text() || 'world';

        return { body: `Hello, ${name}!` };
    }
});
```
This code shows a HTTP POST triggered function which expects `person` object with `name` and `age` values in the request.

```javascript
const { app } = require('@azure/functions');

app.http('httpPostBodyFunction', {
    methods: ['POST'],
    authLevel: 'function',
    handler: async (request, context) => {
        context.log(`Http function processed request for url "${request.url}"`);

        try {
            const person = await request.json();
            const { name, age } = person;

            if (!name || !age) {
                return {
                    status: 400,
                    body: 'Please provide both name and age in the request body.'
                };
            }

            return {
                status: 200,
                body: `Hello, ${name}! You are ${age} years old.`
            };
        } catch (error) {
            return {
                status: 400,
                body: 'Invalid request body. Please provide a valid JSON object with name and age.'
            };
        }
    }
});
```

## Deploy to Azure

To provision the dependent resources and deploy the Function app run the following command:
```bash
azd up
```
You're prompted for an environment name (this is a friendly name for storing azd parameters), your Azure subscription, and the location of an Azure region in which to create the resource group.

## Clean up resources

When you're done with your deployment and to avoid incurring further costs, use the `azd down` command to delete the resource group and all its contained resources that you created using `azd up`.
