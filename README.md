# [Preview] Sample Chat App with AOAI

This repo contains sample code for a simple chat webapp that integrates with Azure OpenAI. Note: some portions of the app use preview APIs.

## Prerequisites
- An existing Azure OpenAI resource and model deployment of a chat model (e.g. `gpt-35-turbo-16k`, `gpt-4`)
- To use Azure OpenAI on your data: an existing Azure Cognitive Search resource and index.

## Deploy the app

### Deploy with Azure Developer CLI
Please see [README_azd.md](./README_azd.md) for detailed instructions.

### One click Azure deployment
[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2Fsample-app-aoai-chatGPT%2Fmain%2Finfrastructure%2Fdeployment.json)

Click on the Deploy to Azure button and configure your settings in the Azure Portal as described in the [Environment variables](#environment-variables) section.

Please see the [section below](#add-an-identity-provider) for important information about adding authentication to your app.

### Deploy from your local machine

#### Local Setup: Basic Chat Experience
1. Update the environment variables listed in `app.py` as described in the [Environment variables](#environment-variables) section.
    
    These variables are required:
    - `AZURE_OPENAI_RESOURCE`
    - `AZURE_OPENAI_MODEL`
    - `AZURE_OPENAI_KEY`

    These variables are optional:
    - `AZURE_OPENAI_TEMPERATURE`
    - `AZURE_OPENAI_TOP_P`
    - `AZURE_OPENAI_MAX_TOKENS`
    - `AZURE_OPENAI_STOP_SEQUENCE`
    - `AZURE_OPENAI_SYSTEM_MESSAGE`

    See the [documentation](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/reference#example-response-2) for more information on these parameters.

2. Start the app with `start.cmd`. This will build the frontend, install backend dependencies, and then start the app.

3. You can see the local running app at http://127.0.0.1:5000.

#### Local Setup: Chat with your data (Preview)
[More information about Azure OpenAI on your data](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/concepts/use-your-data)

1. Update the `AZURE_OPENAI_*` environment variables as described above. 
2. To connect to your data, you need to specify an Azure Cognitive Search index to use. You can [create this index yourself](https://learn.microsoft.com/en-us/azure/search/search-get-started-portal) or use the [Azure AI Studio](https://oai.azure.com/portal/chat) to create the index for you.

    These variables are required when adding your data:
    - `AZURE_SEARCH_SERVICE`
    - `AZURE_SEARCH_INDEX`
    - `AZURE_SEARCH_KEY`

    These variables are optional:
    - `AZURE_SEARCH_USE_SEMANTIC_SEARCH`
    - `AZURE_SEARCH_SEMANTIC_SEARCH_CONFIG`
    - `AZURE_SEARCH_INDEX_TOP_K`
    - `AZURE_SEARCH_ENABLE_IN_DOMAIN`
    - `AZURE_SEARCH_CONTENT_COLUMNS`
    - `AZURE_SEARCH_FILENAME_COLUMN`
    - `AZURE_SEARCH_TITLE_COLUMN`
    - `AZURE_SEARCH_URL_COLUMN`
    - `AZURE_SEARCH_VECTOR_COLUMNS`
    - `AZURE_SEARCH_QUERY_TYPE`
    - `AZURE_SEARCH_PERMITTED_GROUPS_COLUMN`
    - `AZURE_SEARCH_STRICTNESS`
    - `AZURE_OPENAI_EMBEDDING_NAME`

3. Start the app with `start.cmd`. This will build the frontend, install backend dependencies, and then start the app.
4. You can see the local running app at http://127.0.0.1:5000.

#### Local Setup: Enable Chat History
To enable chat history, you will need to set up CosmosDB resources. The ARM template in the `infrastructure` folder can be used to deploy an app service and a CosmosDB with the database and container configured. Then specify these additional environment variables: 
- `AZURE_COSMOSDB_ACCOUNT`
- `AZURE_COSMOSDB_DATABASE`
- `AZURE_COSMOSDB_CONVERSATIONS_CONTAINER`
- `AZURE_COSMOSDB_ACCOUNT_KEY`

As above, start the app with `start.cmd`, then visit the local running app at http://127.0.0.1:5000.

#### Local Setup: Enable Message Feedback
To enable message feedback, you will need to set up CosmosDB resources. Then specify these additional environment variable:

/.env
- `AZURE_COSMOSDB_ENABLE_FEEDBACK=True`

#### Deploy with the Azure CLI
**NOTE**: If you've made code changes, be sure to **build the app code** with `start.cmd` or `start.sh` before you deploy, otherwise your changes will not be picked up. If you've updated any files in the `frontend` folder, make sure you see updates to the files in the `static` folder before you deploy.

You can use the [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli) to deploy the app from your local machine. Make sure you have version 2.48.1 or later.

If this is your first time deploying the app, you can use [az webapp up](https://learn.microsoft.com/en-us/cli/azure/webapp?view=azure-cli-latest#az-webapp-up). Run the following command from the root folder of the repo, updating the placeholder values to your desired app name, resource group, location, and subscription. You can also change the SKU if desired.

`az webapp up --runtime PYTHON:3.10 --sku B1 --name PON-CAT-2 --resource-group PON-CAT --location "West Europe" --subscription "Azure-abonnement 1"`

If you've deployed the app previously, first run this command to update the appsettings to allow local code deployment:

`az webapp config appsettings set -g PON-CAT -n PON-CAT-2 --settings WEBSITE_WEBDEPLOY_USE_SCM=false`

Check the runtime stack for your app by viewing the app service resource in the Azure Portal. If it shows "Python - 3.10", use `PYTHON:3.10` in the runtime argument below. If it shows "Python - 3.11", use `PYTHON:3.11` in the runtime argument below. 

Check the SKU in the same way. Use the abbreviated SKU name in the argument below, e.g. for "Basic (B1)" the SKU is `B1`. 

Then, use the `az webapp up` command to deploy your local code to the existing app:

`az webapp up --runtime PYTHON:3.10 --sku B1 --name PON-CAT-2 --resource-group PON-CAT`

Make sure that the app name and resource group match exactly for the app that was previously deployed.

Deployment will take several minutes. When it completes, you should be able to navigate to your app at {app-name}.azurewebsites.net.

### Add an identity provider
After deployment, you will need to add an identity provider to provide authentication support in your app. See [this tutorial](https://learn.microsoft.com/en-us/azure/app-service/scenario-secure-app-authentication-app-service) for more information.

If you don't add an identity provider, the chat functionality of your app will be blocked to prevent unauthorized access to your resources and data. 

To remove this restriction, you can add `AUTH_ENABLED=False` to the environment variables. This will disable authentication and allow anyone to access the chat functionality of your app. **This is not recommended for production apps.**

To add further access controls, update the logic in `getUserInfoList` in `frontend/src/pages/chat/Chat.tsx`. 

## Common Customization Scenarios
Feel free to fork this repository and make your own modifications to the UX or backend logic. For example, you may want to change aspects of the chat display, or expose some of the settings in `app.py` in the UI for users to try out different behaviors. 

### Scalability
For apps published with `az webapp up` or from the Azure AI Studio, you can increase your app's ability to handle concurrent requests from multiple users with the following steps:
1. Upgrade your App Service plan tier to a higher tier, for example tiers with more than one vCPU.

2. Configure the following app setting on your App Service in the Azure Portal:
- `PYTHON_ENABLE_GUNICORN_MULTIWORKERS`: true
This will default to use a default worker count of (2 * numCores) + 1 and thread count of 1.
If your App Service Plan has additional compute capacity and you want to increase the worker or thread count, you can figure these additional settings accordingly:
- `PYTHON_GUNICORN_CUSTOM_WORKER_NUM`
- `PYTHON_GUNICORN_CUSTOM_THREAD_NUM`

See the [Oryx documentation](https://github.com/microsoft/Oryx/blob/main/doc/configuration.md) for more details on these settings.

After adding the settings, be sure to save the configuration and then restart your app.

For apps published with `One click Azure deployment` you can increase your app's ability to handle concurrent requests from multiple users with the following steps:
1. Upgrade your App Service plan tier to a higher tier, for example tiers with more than one vCPU.

2. Configure the following app settings on your App Service in the Azure Portal:
- `UWSGI_PROCESSES`: 5 (may be higher or lower depending on your App Service Plan tier)
- `UWSGI_THREADS`: 5 (may be higher or lower depending on your App Service Plan tier)

After adding the settings, be sure to save the configuration and then restart your app.

> In case you build your own docker image based on the `WebApp.Dockerfile` and host it in a App Serice, you can also increase your app's ability to handle concurrent requests from multiple users with the above steps.
When you host your container not in an App Service you can add this seetings as container environment variable.

### Debugging your deployed app
First, add an environment variable on the app service resource called "DEBUG". Set this to "true".

Next, enable logging on the app service. Go to "App Service logs" under Monitoring, and change Application logging to File System. Save the change.

Now, you should be able to see logs from your app by viewing "Log stream" under Monitoring.

### Configuring vector search
When using your own data with a vector index, ensure these settings are configured on your app:
- `AZURE_SEARCH_QUERY_TYPE`: can be `vector`, `vectorSimpleHybrid`, or `vectorSemanticHybrid`,
- `AZURE_OPENAI_EMBEDDING_NAME`: the name of your Ada (text-embedding-ada-002) model deployment on your Azure OpenAI resource.
- `AZURE_SEARCH_VECTOR_COLUMNS`: the vector columns in your index to use when searching. Join them with `|` like `contentVector|titleVector`.

### Updating the default chat logo and headers
The landing chat page logo and headers are specified in `frontend/src/pages/chat/Chat.tsx`:
```
<Stack className={styles.chatEmptyState}>
    <img
        src={Azure}
        className={styles.chatIcon}
        aria-hidden="true"
    />
    <h1 className={styles.chatEmptyStateTitle}>Start chatting</h1>
    <h2 className={styles.chatEmptyStateSubtitle}>This chatbot is configured to answer your questions</h2>
</Stack>
```
To update the logo, change `src={Azure}` to point to your own SVG file, which you can put in `frontend/src/assets`/
To update the headers, change the strings "Start chatting" and "This chatbot is configured to answer your questions" to your desired values.

### Changing Citation Display
The Citation panel is defined at the end of `frontend/src/pages/chat/Chat.tsx`. The citations returned from Azure OpenAI On Your Data will include `content`, `title`, `filepath`, and in some cases `url`. You can customize the Citation section to use and display these as you like. For example, the title element is a clickable hyperlink if `url` is not a blob URL.

```
    <h5 
        className={styles.citationPanelTitle} 
        tabIndex={0} 
        title={activeCitation.url && !activeCitation.url.includes("blob.core") ? activeCitation.url : activeCitation.title ?? ""} 
        onClick={() => onViewSource(activeCitation)}
    >{activeCitation.title}</h5>

    const onViewSource = (citation: Citation) => {
        if (citation.url && !citation.url.includes("blob.core")) {
            window.open(citation.url, "_blank");
        }
    };

```


### Best Practices
We recommend keeping these best practices in mind:

- Reset the chat session (clear chat) if the user changes any settings. Notify the user that their chat history will be lost.
- Clearly communicate to the user what impact each setting will have on their experience.
- When you rotate API keys for your AOAI or ACS resource, be sure to update the app settings for each of your deployed apps to use the new key.
- Pull in changes from `main` frequently to ensure you have the latest bug fixes and improvements, especially when using Azure OpenAI on your data.

## Environment variables

Note: settings starting with `AZURE_SEARCH` are only needed when using Azure OpenAI on your data. If not connecting to your data, you only need to specify `AZURE_OPENAI` settings.

| App Setting | Value | Note |
| --- | --- | ------------- |
|AZURE_SEARCH_SERVICE||The name of your Azure Cognitive Search resource|
|AZURE_SEARCH_INDEX||The name of your Azure Cognitive Search Index|
|AZURE_SEARCH_KEY||An **admin key** for your Azure Cognitive Search resource|
|AZURE_SEARCH_USE_SEMANTIC_SEARCH|False|Whether or not to use semantic search|
|AZURE_SEARCH_QUERY_TYPE|simple|Query type: simple, semantic, vector, vectorSimpleHybrid, or vectorSemanticHybrid. Takes precedence over AZURE_SEARCH_USE_SEMANTIC_SEARCH|
|AZURE_SEARCH_SEMANTIC_SEARCH_CONFIG||The name of the semantic search configuration to use if using semantic search.|
|AZURE_SEARCH_TOP_K|5|The number of documents to retrieve from Azure Cognitive Search.|
|AZURE_SEARCH_ENABLE_IN_DOMAIN|True|Limits responses to only queries relating to your data.|
|AZURE_SEARCH_CONTENT_COLUMNS||List of fields in your Azure Cognitive Search index that contains the text content of your documents to use when formulating a bot response. Represent these as a string joined with "|", e.g. `"product_description|product_manual"`|
|AZURE_SEARCH_FILENAME_COLUMN|| Field from your Azure Cognitive Search index that gives a unique idenitfier of the source of your data to display in the UI.|
|AZURE_SEARCH_TITLE_COLUMN||Field from your Azure Cognitive Search index that gives a relevant title or header for your data content to display in the UI.|
|AZURE_SEARCH_URL_COLUMN||Field from your Azure Cognitive Search index that contains a URL for the document, e.g. an Azure Blob Storage URI. This value is not currently used.|
|AZURE_SEARCH_VECTOR_COLUMNS||List of fields in your Azure Cognitive Search index that contain vector embeddings of your documents to use when formulating a bot response. Represent these as a string joined with "|", e.g. `"product_description|product_manual"`|
|AZURE_SEARCH_PERMITTED_GROUPS_COLUMN||Field from your Azure Cognitive Search index that contains AAD group IDs that determine document-level access control.|
|AZURE_SEARCH_STRICTNESS|3|Integer from 1 to 5 specifying the strictness for the model limiting responses to your data.|
|AZURE_OPENAI_RESOURCE||the name of your Azure OpenAI resource|
|AZURE_OPENAI_MODEL||The name of your model deployment|
|AZURE_OPENAI_ENDPOINT||The endpoint of your Azure OpenAI resource.|
|AZURE_OPENAI_MODEL_NAME|gpt-35-turbo-16k|The name of the model|
|AZURE_OPENAI_KEY||One of the API keys of your Azure OpenAI resource|
|AZURE_OPENAI_TEMPERATURE|0|What sampling temperature to use, between 0 and 2. Higher values like 0.8 will make the output more random, while lower values like 0.2 will make it more focused and deterministic. A value of 0 is recommended when using your data.|
|AZURE_OPENAI_TOP_P|1.0|An alternative to sampling with temperature, called nucleus sampling, where the model considers the results of the tokens with top_p probability mass. We recommend setting this to 1.0 when using your data.|
|AZURE_OPENAI_MAX_TOKENS|1000|The maximum number of tokens allowed for the generated answer.|
|AZURE_OPENAI_STOP_SEQUENCE||Up to 4 sequences where the API will stop generating further tokens. Represent these as a string joined with "|", e.g. `"stop1|stop2|stop3"`|
|AZURE_OPENAI_SYSTEM_MESSAGE|You are an AI assistant that helps people find information.|A brief description of the role and tone the model should use|
|AZURE_OPENAI_PREVIEW_API_VERSION|2023-06-01-preview|API version when using Azure OpenAI on your data|
|AZURE_OPENAI_STREAM|True|Whether or not to use streaming for the response|
|AZURE_OPENAI_EMBEDDING_NAME||The name of your embedding model deployment if using vector search.


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
