
# IMT - Veritone Integration Guide

Sections:

[creating a job](#creating-a-job)
  
  - [transcription job](#transcription)
  
  - [facial recognition job](#facial-recognition)
  
  - [logo recognition job](#logo-recognition)
  
  - [google ocr job](#google-ocr)
  
[checking job status](#checking-job-status)

[retrieving job output](#retrieving-job-output)

[job notifications](#job-notifications)

## Veritone API Flow

1. `createJob` - Create a cognition job.
2. `checkJobStatus` - Check the status of a job. (optional, you can also use a webhook)
3. `retrieveEngineOutput` - Retrieve the output of a completed job.


## Using Veritone's API

### Set your request headers

Setting a header "x-veritone-application":"org:orgGuid" allows Veritone to monitor API usage and provides better tracking for debugging purposes.  

Example: `"x-veritone-application": "org:your_org_guid"`

Please set `x-veritone-application` header on all API requests. If you do not know your orgGuid, contact your customer service manager. 


## Example Veritone API Calls

# Creating a job

Choose a template from the options below that fits your desired job type.  There are two necessary items for each job.

- `sourceUrl` A link to your file (signedUrl).

- `engineId` Id of your desired cognition engine.

For a more in-depth look at Job creation, visit the official [Veritone documentation](https://docs.veritone.com/#/quickstart/jobs/?id=working-with-jobs).

# Transcription

Transcription engineId(s):

 Engine Name             | engineId                             | payload name
 ----------------------- | ------------------------------------ | --------------------------
 Speechmatics English    | c0e55cde-340b-44d7-bb42-2e0d65e98255 | keywords (optional)
 Speechmatics Spanish.   | 71ab1ba9-e0b8-4215-b4f9-0fc1a1d2b255 | keywords (optional)


### Transcription template

This mutation returns a jobId and a TDOId.  These values are used when retrieving information about the job.  The TDOId holds all the information on a file and can have multiple jobs associated with it.  When you first create a job, a jobId and TDOId will be generated for you.  Subsequent job requests on the same file can be made to the TDOId.

```
mutation TranscriptionJob {
    createJob(input: {
      target: { status: "downloaded" }
      tasks: [
        {
          engineId: "8bdb0e3b-ff28-4f6e-a3ba-887bd06e6440"
          payload:{
            ffmpegTemplate: "audio",
            url: "LINK_TO_YOUR_FILE",
            customFFMPEGProperties: { chunkSizeInSeconds: "300" }
          }
          ioFolders: [
            { referenceId: "siOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: false, priority:-20 }
        }
        {
          engineId: "ENGINE_ID"
          # payload: { keywords: "" }
          ioFolders: [
            { referenceId: "engineInputFolder", mode: chunk, type: input }
            { referenceId: "engineOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
        }
        {
          # Output writer
          engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
          ioFolders: [
            { referenceId: "owInputFolder", mode: chunk, type: input }
          ]
        }
      ]
      routes: [
        { 
          # chunker --> engine
          parentIoFolderReferenceId: "siOutputFolder"
          childIoFolderReferenceId: "engineInputFolder"
          options: {}
        }
        { 
          # engine --> output writer
          parentIoFolderReferenceId: "engineOutputFolder"
          childIoFolderReferenceId: "owInputFolder"
          options: {}
        }
      ]
    }){
      targetId
      id
}}
```

# Facial Recognition

FaceBox engineId(s):

 Engine Name             | engineId                             | payload name
 ----------------------- | ------------------------------------ | --------------------------
 FaceBox Recognize       | e62665c7-f855-4168-8aa3-668a7b0a50ea | libraryId


NOTE: Facial recognition requires a trained library in order to successfully process media!

```
mutation FaceBoxJob {
    createJob(input: {
      target: { status: "downloaded" } 
      tasks: [
        {
          engineId: "8bdb0e3b-ff28-4f6e-a3ba-887bd06e6440"
          payload:{
            ffmpegTemplate: "frame",
            url: "LINK_TO_YOUR_FILE",
            customFFMPEGProperties: { framesPerSecond: "1" }
          }
          ioFolders: [
            { referenceId: "siOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: false, priority:-20 }
        }
        {
          engineId: "ENGINE_ID"
          payload: {
            mode: "library-run",
            libraryId: "YOUR_LIBRARY_ID",
          }          
          ioFolders: [
            { referenceId: "engineInputFolder", mode: chunk, type: input }
            { referenceId: "engineOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
        }
        {
          # Output writer
          engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
          ioFolders: [
            { referenceId: "owInputFolder", mode: chunk, type: input }
          ]
        }
      ]
      routes: [
        { 
          # chunker --> engine
          parentIoFolderReferenceId: "siOutputFolder"
          childIoFolderReferenceId: "engineInputFolder"
          options: {}
        }
        { 
          # engine --> output writer
          parentIoFolderReferenceId: "engineOutputFolder"
          childIoFolderReferenceId: "owInputFolder"
          options: {}
        }
      ]
    }){
      targetId
      id
}}
```

# Logo Recognition

Logo Recognition engineId(s):

 Engine Name             | engineId                             | payload name
 ----------------------- | ------------------------------------ | --------------------------
 LogoGrab (low res)      | 4a52a78f-3bc7-4b3a-9b5a-1a192cc6cfe1 | key


```
mutation LogoRecognitionJob {
    createJob(input: {
      target: { status: "downloaded" } 
      tasks: [
        {
          engineId: "8bdb0e3b-ff28-4f6e-a3ba-887bd06e6440"
          payload:{
            ffmpegTemplate: "frame",
            url: "LINK_TO_YOUR_FILE",
            customFFMPEGProperties: { framesPerSecond: "1" }
          }
          ioFolders: [
            { referenceId: "siOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: false, priority:-20 }
        }
        {
          engineId: "ENGINE_ID"
          payload: { key: "1arju7us157v6cldj6u6bdd7bnhg5o8095dru7nh" }        
          ioFolders: [
            { referenceId: "engineInputFolder", mode: chunk, type: input }
            { referenceId: "engineOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
        }
        {
          # Output writer
          engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
          ioFolders: [
            { referenceId: "owInputFolder", mode: chunk, type: input }
          ]
        }
      ]
      routes: [
        { 
          # chunker --> engine
          parentIoFolderReferenceId: "siOutputFolder"
          childIoFolderReferenceId: "engineInputFolder"
          options: {}
        }
        { 
          # engine --> output writer
          parentIoFolderReferenceId: "engineOutputFolder"
          childIoFolderReferenceId: "owInputFolder"
          options: {}
        }
      ]
    }){
      targetId
      id
}}
```

# Google OCR

Google OCR engineId(s):

 Engine Name             | engineId                             | payload name
 ----------------------- | ------------------------------------ | --------------------------
 Google OCR              | 1e5b4e36-acbe-49f1-bebf-1d3a0edd6219 | n/a

```
mutation GoogleOCRJob {
    createJob(input: {
      target: { status: "downloaded" } 
      tasks: [
        {
          engineId: "8bdb0e3b-ff28-4f6e-a3ba-887bd06e6440"
          payload:{
            ffmpegTemplate: "frame",
            url: "LINK_TO_YOUR_FILE",
            customFFMPEGProperties: { framesPerSecond: "1" }
          }
          ioFolders: [
            { referenceId: "siOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: false, priority:-20 }
        }
        {
          engineId: "ENGINE_ID"    
          ioFolders: [
            { referenceId: "engineInputFolder", mode: chunk, type: input }
            { referenceId: "engineOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
        }
        {
          # Output writer
          engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
          ioFolders: [
            { referenceId: "owInputFolder", mode: chunk, type: input }
          ]
        }
      ]
      routes: [
        { 
          # chunker --> engine
          parentIoFolderReferenceId: "siOutputFolder"
          childIoFolderReferenceId: "engineInputFolder"
          options: {}
        }
        { 
          # engine --> output writer
          parentIoFolderReferenceId: "engineOutputFolder"
          childIoFolderReferenceId: "owInputFolder"
          options: {}
        }
      ]
    }){
      targetId
      id
}}
```

# Checking job status

You can poll for job status updates or use webhook notifications. (detailed below)

```
query queryJobStatus {
  job(id:"JOB_ID") {
    id
    status
    targetId
    tasks{
      records{
        id
        status
        engine{
          id
          name
        }
        # taskOutput - include this field for a more verbose output
      }
    }
  }
}
```
# Retrieving job output

Include the TDO ID and Engine ID to retrieve transcription output in JSON format.

```
query getEngineOutput {
  engineResults(tdoId: "TDO_ID",
    engineIds: ["ENGINE_ID"]) { # Retrieves results from specified engine
    records {
      tdoId
      engineId
      startOffsetMs
      stopOffsetMs
      jsondata
      assetId
      userEdited
}}}
```
# Job Notifications

Veritone's `notificationUris` field allows you to link to a custom endpoint(s).  This removes the need to poll for job completion, you will instead be notified when a job or task completes at the endpoint provided.

This example notifies you when the output is completed:

```
{
  # Output writer
  engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
  executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
  ioFolders: [
    { referenceId: "owInputFolder", mode: chunk, type: input }
  ]
  notificationUris: ["https://example.net/dump"]
}
```
