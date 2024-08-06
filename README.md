---
name: "Azure Functions JavaScript HTTP Trigger using AZD"
description: This repository contains a Azure Functions HTTP trigger quickstart written in JavaScript and deployed to Azure using Azure Developer CLI (AZD). This sample uses manaaged identity and a virtual network to insure it is secure by default.
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

This repository contains a Azure Functions HTTP trigger quickstart written in JavaScript and deployed to Azure using Azure Developer CLI (AZD). This sample uses manaaged identity and a virtual network to insure it is secure by default. 

## Getting Started

### Prerequisites

1) [Node.js 20](https://www.nodejs.org/) 
2) [Azure Functions Core Tools](https://learn.microsoft.com/azure/azure-functions/functions-run-local?tabs=v4%2Cmacos%2Ccsharp%2Cportal%2Cbash#install-the-azure-functions-core-tools)
3) [Azure Developer CLI (AZD)](https://learn.microsoft.com/azure/developer/azure-developer-cli/install-azd)
4) [Visual Studio Code](https://code.visualstudio.com/) - Only required if using VS Code to run locally

### Get repo on your local machine
Run the following GIT command to clone this repository to your local machine.
```bash
git clone https://github.com/Azure-Samples/functions-quickstart-javascript-azd.git
```

## Run on your local environment

### Prepare your local environment
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
1) Open this folder in a new terminal and run the following commands:

```bash
npm install
func start
```

2) Test the HTTP GET trigger using the browser to open http://localhost:7071/api/httpGetFunction

3) Test the HTTP POST trigger in a new terminal window with the following command, or use your favorite REST client, e.g. [RestClient in VS Code](https://marketplace.visualstudio.com/items?itemName=humao.rest-client), PostMan, curl. `test.http` has been provided to run this quickly.

```bash
curl -i -X POST http://localhost:7071/api/httppostbodyfunction -H "Content-Type: text/json" --data-binary "@src/functions/testdata.json"
```

### Using Visual Studio Code
1) Open this folder in a new terminal
2) Open VS Code by entering `code .` in the terminal
3) Make sure the Azure Functions extension is installed
4) Add a **.vscode** folder by running *"Azure Functions: Initialize project for use with VS Code"* in the Command Pallete
5) Press Run/Debug (F5) to run in the debugger (select "Debug anyway" if prompted about local emulater not running) 
6) Use same approach above to test using an HTTP REST client

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
You will be prompted for an environment name (this is a friendly name for storing AZD parameters), a Azure subscription, and an Aure location.