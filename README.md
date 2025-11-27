Event Grid: Auto-Index Blob Metadata into Cosmos DB

Project Overview
Automatically indexes newly uploaded blobs into Cosmos DB.
Uses Event Grid to trigger an Azure Function whenever a blob is created in the documents container.
Extracts metadata and content, including title and word count for text files.
Inserts the extracted data into Cosmos DB, avoiding duplicate inserts.
Workflow
Event Grid Trigger fires when a blob is created in the storage account.
Azure Function Execution:
Extracts blob metadata: blobName, container, url, size, contentType.
Reads content for text files: title (first line or H1), wordCount.
Insert into Cosmos DB: Adds or updates document in Documents container with id=blobName.
Logging: Logs operations; sample query can fetch largest documents.

Azure Components Used
Azure Storage Account – Source of blobs.
Event Grid – Triggers function on blob creation.
Azure Functions – Processes blob metadata and content.
Cosmos DB – Stores indexed documents.
Application Insights / Blob Logger – Tracks operations.

Tech Stack
Python
Azure Functions
Azure Storage Blob
Event Grid
Azure Cosmos DB
Local Development Setup
Create an Azure Functions project in VS Code.

Install required Python libraries:
azure-functions
azure-storage-blob
azure-cosmos
Configure local.settings.json:
FUNCTIONS_WORKER_RUNTIME
AzureWebJobsStorage
COSMOS_ENDPOINT
COSMOS_KEY
COSMOS_DB
COSMOS_CONTAINER

Run locally using:
func start
Function Behavior / Testing
Function triggers automatically on blob upload.
Processes text files for title and word count.
Inserts document into Cosmos DB; duplicates handled safely with upsert.
Logs show processed blobs and inserted documents.

Sample Cosmos Document
{
  "id": "file1.txt",
  "url": "https://.../documents/file1.txt",
  "size": 1234,
  "title": "My Notes",
  "wordCount": 345,
  "uploadedOn": "2025-11-24T..."
}

Sample Cosmos DB Query

To get largest documents:

SELECT c.id, c.size 
FROM c 
ORDER BY c.size DESC

Deployment Steps
Open VS Code → Azure Extension → Deploy to Function App.
Select your subscription and Function App.
Configure AzureWebJobsStorage and Cosmos DB settings in Application Settings.
Restart the Function App.

Notes
Detect text files via contentType (text/*) or file extension .txt.
Use Event Grid event data URL to download blobs.
Safe upsert prevents duplicate documents.

Evaluation
Event triggers are delivered successfully.
Documents appear correctly in Cosmos DB
Duplicate uploads either update or safely upsert existing documents.
