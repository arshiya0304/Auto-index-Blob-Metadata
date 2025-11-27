Overviews
An Event Grid-triggered Function listens for blob-created events on a storage account; for each new blob in documents container it extracts metadata and content, computes title (first H1 or first line) and wordCount (for text files), and upserts a document into Cosmos DB Documents container with id = blobName.

Requirements / Acceptance
Function triggered by Event Grid blob-created events.
Extract: blobName, container, url, size, contentType, title, wordCount.
Insert into Cosmos DB with id=blobName, avoid duplicate inserts (use Upsert).
Provide example query to get largest documents.
Architecture
Event Grid (Blob created) -> Function (EventGridTrigger) -> Blob Storage read -> Cosmos DB Upsert.

Tech Stack
Python / Node / C# (examples use Python here)
azure-functions
azure-storage-blob
azure-cosmos
Setup

Install deps
pip install azure-functions azure-storage-blob azure-cosmos
local.settings.json:

{
  "Values": {
    "AzureWebJobsStorage": "<conn>",
    "COSMOS_ENDPOINT": "https://...",
    "COSMOS_KEY": "...",
    "COSMOS_DATABASE": "DocumentsDB",
    "COSMOS_CONTAINER": "Documents"
  }
}
Function Behavior

On event, use data.url to download blob via anonymous or SAS URL, or use SDK with container+blob name.
For text files (contentType starts with text/ or suffix in .txt,.md,.csv,.html):
Read first line or parse for first H1 (# or <h1>), set as title.
Count words by splitting on whitespace to produce wordCount.
For non-text files: set title to filename (or metadata title if present) and wordCount = 0.

Create a Cosmos document:

{
  "id": "file1.txt",
  "url": "https://.../documents/file1.txt",
  "size": 1234,
  "title": "My Notes",
  "wordCount": 345,
  "uploadedOn": "2025-11-24T..."
}

Use upsert_item to avoid duplicate inserts; optionally perform a pre-check or rely on Cosmos concurrency.

Sample Cosmos Query (largest docs)
SELECT TOP 10 c.id, c.size, c.title
FROM c
ORDER BY c.size DESC
Testing
Upload a text blob to documents container and verify Cosmos contains a document with correct fields.
Re-upload same blob name to verify upsert behaviour.

Troubleshooting
Event Grid subscription not firing: ensure event subscription is configured for blob-created and correct resource.
Authentication failures: validate keys and RBAC for the Function's identity.
