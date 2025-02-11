---
title: Fabric data pipeline public REST API (Preview)
description: This article describes the available REST APIs for pipelines in Data Factory for Microsoft Fabric.
author: kromerm
ms.author: makromer
ms.topic: conceptual
ms.date: 09/16/2024
---

# Microsoft Fabric data pipeline public REST API (Preview)

> [!IMPORTANT]
> The Microsoft Fabric API for Data Factory is currently in public preview. This information relates to a prerelease product that may be substantially modified before it's released. Microsoft makes no warranties, expressed or implied, with respect to the information provided here.

In Microsoft Fabric, Data Factory APIs consist solely of CRUD operations for pipelines and dataflows. Currently, only data pipelines are supported. Dataflows APIs will be released later. Other common areas for data integration projects are in separate APIs: schedules, monitoring, connections, have their own APIs in Fabric. The primary online reference documentation for Microsoft Fabric REST APIs can be found in [Microsoft Fabric REST API references](/rest/api/fabric/articles/). Also refer to the [Core items API](/rest/api/fabric/core/items) and [Job scheduler](/rest/api/fabric/core/job-scheduler).

## Mounting Public APIs
The Mounting Public APIs are now available. These APIs allow you to seamlessly integrate and access various public data sources within your data pipelines.

## Obtain an authorization token

### Option 1: Using MSAL.Net

[Fabric API quickstart - Microsoft Fabric REST APIs](/rest/api/fabric/articles/get-started/fabric-api-quickstart#get-token)

Use MSAL.Net to acquire a Microsoft Entra ID token for Fabric service with the following scopes: Workspace.ReadWrite.All, Item.ReadWrite.All. For more information about token acquisition with MSAL.Net to, see [Token Acquisition - Microsoft Authentication Library for .NET](/entra/msal/dotnet/acquiring-tokens/overview).

Paste the Application (client) ID you copied earlier and paste it for ClientId variable.

### Option 2: Using the Fabric Portal

Sign in into the Fabric Portal for the Tenant you want to test on, and press F12 to enter the browser's developer mode. In the console there, run:

```powerBIAccessToken```

Copy the token and paste it for the ClientId variable.

## Item Definition with payload base64 encoded

- Use [Base64 Encode and Decode](https://www.base64encode.org/) to encode your JSON.
- Ensure that the **Perform URL safe encoding** box isn't checked.
- You can get the pipeline definitions via the **View** --> **View JSON code** tab in the Fabric user interface.

```json
{ 
    "name": "Pipeline_1_updated", 
    "objectId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", 
    "properties": { 
        "description": "this is the description", 
        "activities": [ 
            { 
                "name": "Wait1", 
                "type": "Wait", 
                "dependsOn": [], 
                "typeProperties": { 
                    "waitTimeInSeconds": 240 
                } 
            } 
        ], 
        "annotations": [], 
        "lastModifiedByObjectId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", 
        "lastPublishTime": "2024-02-01T17:28:02Z" 
    } 
}
```

Take the properties object and surround them in braces - **{ }** - so the REST Item definition payload would be:

```json
{
    "properties": { 
        "description": "this is the description", 
        "activities": [ 
            { 
                "name": "Wait1", 
                "type": "Wait", 
                "dependsOn": [], 
                "typeProperties": { 
                    "waitTimeInSeconds": 240 
                } 
            } 
        ], 
        "annotations": [], 
        "lastModifiedByObjectId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", 
        "lastPublishTime": "2024-02-01T17:28:02Z" 
    } 
} 
```

## Create item

[REST API - Items - Create item](/rest/api/fabric/core/items/create-item )

Example 1 - CreateDataPipeline:

```POST https://api.fabric.microsoft.com/v1/workspaces/<your WS Id>/items```

Body: 

```json
{ 
  "displayName": "pipeline_1", 
  "type": "DataPipeline" 
} 
```

> [!NOTE]
> The documentation states that there are only 2 required properties - **displayName** and **type**. Currently, Workload-DI does not support creation without a **definition** as well. The fix for this erroneous requirement is currently being deployed. For now, you can send the same default definition used by the Fabric user interface:
> ```‘{"properties":{"activities":[]}}’```

Modified JSON including definition:

```json
{ 
  "displayName": "pipeline_1", 
  "type": "DataPipeline", 
  "definition": { 
    "parts": [ 
      { 
        "path": "pipeline-content.json", 
        "payload": "eyJwcm9wZXJ0aWVzIjp7ImFjdGl2aXRpZXMiOltdfX0=", 
        "payloadType": "InlineBase64" 
      } 
    ] 
  } 
} 
```

Response 201:

```json
{ 
    "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", 
    "type": "DataPipeline", 
    "displayName": "Pipeline_1", 
    "description": "", 
    "workspaceId": "<Your WS Id>" 
} 
```

Example 2 – Create MountedDataFactory 

```POST https://api.fabric.microsoft.com/v1/workspaces/<your WS Id>/items```

Body: 

Payload:

```json
{"DataFactoryResourceId":"/subscriptions/<ADF subscription Id>/resourceGroups/<ADF resource group name>/providers/Microsoft.DataFactory/factories/<ADF datafactory name>"} 
```

Encoded JSON:

```json
{ 
  "displayName": "pipeline_mdf", 
  "type": " MountedDataFactory ", 
  "definition": { 
    "parts": [ 
      { 
        "path": "mountedDataFactory-content.json", 
        "payload": <base64 encoded value>, 
        "payloadType": "InlineBase64" 
      } 
    ] 
  } 
}  
```

Response 201:

```json
{ 
    "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", 
    "type": "MountedDataFactory", 
    "displayName": "Pipeline_mdf", 
    "description": "", 
    "workspaceId": "<Your WS Id>"
} 
```

## Delete item

[REST API - Items - Delete item](/rest/api/fabric/core/items/delete-item)

Example:

```DELETE https://api.fabric.microsoft.com/v1/workspaces/<your WS Id>/items/<pipeline id> ```

Response 200: (No body)

## Get item

[REST API - Items - Get item](/rest/api/fabric/core/items/get-item)

Example:

```GET https://api.fabric.microsoft.com/v1/workspaces/<your WS Id>/items/<pipeline id>```

Response 200:

```json
{ 
    "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", 
    "type": "DataPipeline", 
    "displayName": "Pipeline_1", 
    "workspaceId": "<your WS Id>" 
} 
```

## Get item definition

[REST API - Items - Get item definition](/rest/api/fabric/core/items/get-item-definition)

Example:

```POST https://api.fabric.microsoft.com/v1/workspaces/<your WS Id>/items/<pipeline id>/getDefinition```

Response 200:

```json
{ 
    "definition": { 
        "parts":[ 
            { 
                "path": "pipeline-content.json", 
                "payload": "ewogICJwcm9wZXJ0aWVzIjogewogICAgImFjdGl2aXRpZXMiOiBbXQogIH0KfQ==", 
                "payloadType": "InlineBase64" 
            } 
        ] 
    } 
} 
```

## List items

[REST API - Items - List items](/rest/api/fabric/core/items/list-items)

Example:

```GET https://api.fabric.microsoft.com/v1/workspaces/<your WS Id>/items```

Response 200:

```json
{ 
    "value": [ 
        { 
            "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", 
            "type": "SemanticModel", 
            "displayName": "deata_lh", 
            "description": "", 
            "workspaceId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" 
        }, 
        { 
            "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", 
            "type": "SQLEndpoint", 
            "displayName": "deata_lh", 
            "description": "", 
            "workspaceId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" 
        }, 
        { 
            "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", 
            "type": "Lakehouse", 
            "displayName": "deata_lh", 
            "description": "", 
            "workspaceId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" 
        }, 
        { 
            "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", 
            "type": "DataPipeline", 
            "displayName": "Pipeline_1", 
            "description": "", 
            "workspaceId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" 
        } 
    ] 
} 
```

## Update item

[REST API - Items - Update item](/rest/api/fabric/core/items/update-item)

Example:

```PATCH https://api.fabric.microsoft.com/v1/workspaces/<your WS Id>/items/<pipeline id>```

Body:
```json
{ 
  "displayName": "Pipeline_1_updated", 
  "description": "This is the description." 
}
```

Response 200:

```json
{ 
    "id": "<pipeline id>", 
    "type": "DataPipeline", 
    "displayName": "Pipeline_1_updated", 
    "description": "This is the description.", 
    "workspaceId": "<Your WS id>" 
}
```

## Update item definition

[REST API - Items - Update item definition](/rest/api/fabric/core/items/update-item-definition)

Example:

```POST https://api.fabric.microsoft.com/v1/workspaces/<your WS Id>/items/<pipeline id>/updateDefinition```

Body:

```json
{ 
  "definition": { 
    "parts": [ 
      { 
        "path": "pipeline-content.json", 
        "payload": "eyJwcm9wZXJ0aWVzIjp7ImFjdGl2aXRpZXMiOltdfX0=", 
        "payloadType": "InlineBase64" 
      } 
    ] 
  } 
}
```

Response 200: (No body)

## Run on-demand item job

[REST API - Items - Run on-demand item job](/rest/api/fabric/core/job-scheduler/run-on-demand-item-job)

Example:

```POST https://api.fabric.microsoft.com/v1/workspaces/<your WS Id>/items/<pipeline id>/jobs/instances?jobType=Pipeline```

Response 202: (No body)

Example with two parameter values:

Here we have a **Wait** activity with a parameter named **param_waitsec** to specify the number of seconds to wait.

```POST https://api.fabric.microsoft.com/v1/workspaces/<your WS Id>/items/<pipeline id>/jobs/instances?jobType=Pipeline```

Body:

```json
{ 
  "executionData": { 
    "parameters": {
      "param_waitsec": "10" 
    } 
  } 
}
```

Response 202: (No body)

> [!NOTE]
> There is no body returned currently, but the job Id should be returned. During the preview, it can be found in the returned headers, in the ‘Location’ property.

## Get item job instance

[REST API - Items - Get item job instance](/rest/api/fabric/core/job-scheduler/get-item-job-instance)

Example:

```GET https://api.fabric.microsoft.com/v1/workspaces/<your WS Id>/items/<pipeline id>/jobs/instances/<job ID>```

Response 200:

```json
{ 
    "id": "4511ffcd-a9f6-4f75-91a9-9ceab08d7539", 
    "itemId": "2bb9fe4a-0a84-4725-a01f-7ac4e6850259", 
    "jobType": "Pipeline", 
    "invokeType": "Manual", 
    "status": "Completed", 
    "failureReason": null, 
    "rootActivityId": "f14bdd95-2cff-4451-b839-bea81509126d", 
    "startTimeUtc": "2024-02-01T03:03:19.8361605", 
    "endTimeUtc": "2024-02-01T03:05:00.3433333" 
} 
```

## Cancel item job instance

[REST API - Items - Cancel item job instance](/rest/api/fabric/core/job-scheduler/cancel-item-job-instance)

Example:

```POST https://api.fabric.microsoft.com/v1/workspaces/<your WS Id>/items/<pipeline id>/jobs/instances/<job ID>/cancel```

Response 202: (No body)

> [!NOTE]
> After cancelling a job you can check the status either by calling **Get item job instance** or looking at the **View run history** in the Fabric user interface.


## Query activity runs

Example:

```POST https://api.fabric.microsoft.com/v1/workspaces/<your WS Id>/datapipelines/pipelineruns/<job id>/queryactivityruns```

Body:

```json
{
  "filters":[],
  "orderBy":[{"orderBy":"ActivityRunStart","order":"DESC"}],
  "lastUpdatedAfter":"2024-05-22T14:02:04.1423888Z",
  "lastUpdatedBefore":"2024-05-24T13:21:27.738Z"
}
```

> [!NOTE]
> "job id" is the same id created and used in the Job Scheduler Public APIs

Response 200:
```json
[
    {
        "pipelineName": "ca91f97e-5bdd-4fe1-b39a-1f134f26a701",
        "pipelineRunId": "f2fa7a0e-586d-4d73-a2b4-7ddc785243ae",
        "activityName": "Wait1",
        "activityType": "Wait",
        "activityRunId": "ef579d3d-d23e-477a-8150-d6e15d66a532",
        "linkedServiceName": "",
        "status": "Succeeded",
        "activityRunStart": "2024-05-23T13:43:03.6397566Z",
        "activityRunEnd": "2024-05-23T13:43:31.3906179Z",
        "durationInMs": 27750,
        "input": {
            "waitTimeInSeconds": 27
        },
        "output": {},
        "error": {
            "errorCode": "",
            "message": "",
            "failureType": "",
            "target": "Wait1",
            "details": ""
        },
        "retryAttempt": null,
        "iterationHash": "",
        "userProperties": {},
        "recoveryStatus": "None",
        "integrationRuntimeNames": null,
        "executionDetails": null,
        "id": "/SUBSCRIPTIONS/4DA86268-68B8-4B08-AD58-A7AEE54138CD/RESOURCEGROUPS/4DA86268-68B8-4B08-AD58-A7AEE54138CD/PROVIDERS/MICROSOFT.TRIDENT/WORKSPACES/4DA86268-68B8-4B08-AD58-A7AEE54138CD/pipelineruns/f2fa7a0e-586d-4d73-a2b4-7ddc785243ae/activityruns/ef579d3d-d23e-477a-8150-d6e15d66a532"
    }
]

```

## Known limitations

- Service Principal Auth (SPN) is currently not supported.

## Related content

- [REST API - Items](/rest/api/fabric/core/items)
- [Use the Fabric Monitoring hub](/admin/monitoring-hub.md)
