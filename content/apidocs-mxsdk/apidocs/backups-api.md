---
title: "Backups API V2"
parent: "deploy-api"
description: "An API to allow the triggering of backups creation, restore, download and to get information about existing snapshots."
menu_order: 10
---

## 1 Introduction

The **Backups API V2** allows you to manage backups of the data in your app hosted in the Mendix Cloud V4.

If you want to download a backup of your data, you need to perform three steps:

1. Create a *snapshot*, or find the `snapshot-id` of an existing one
2. Create an *archive* file from a snapshot
3. Download the archive using the URL provided 

Data **snapshots** consist of a PostgreSQL database dump and all the file objects referenced from the database.

Database **archives** are a zip file which contains all the data in the snapshot, or the database and files separately if you prefer. 

You cannot currently upload an archive through this API. This function is currently only supported via the [Developer Portal](/developerportal/operate/backups). However, you can use this API to restore data from an existing environment snapshot.

V2 of this API is focused on working with snapshots and archives asynchronously, as these can be very long-running tasks for large quantities of data. The [older V1 API](backups-api-v1), in contrast, works synchronously. 

{{% alert type="info" %}}
This article is only applicable to applications deployed in **Mendix Cloud V4**. You can check which version of the Mendix Cloud you are using in the [Developer Portal](/developerportal/deploy/environments-details).
{{% /alert %}}

## 2 Authentication

The Backups API requires authentication via API keys that are bound to your Mendix account (for more information, see [Deploy Authentication](deploy-api#authentication)). In addition to the **API Access** permission, the **Backups** permission is also required to manage backups.

## 3 API Calls

### 3.1 List Environment Snapshots

#### 3.1.1 Description

Lists the snapshots of an environments. By setting the `offset` parameter, you can page through the list of snapshots created for an environment.

```bash
HTTP Method: GET
URL: https://deploy.mendix.com/api/v2/apps/<ProjectId>/environments/<EnvironmentId>/snapshots
```

#### 3.1.2 Request

**Request Parameters**

*   _ProjectId_ (String): Unique project identifier. Can be looked up via the [developer portal](/developerportal/deploy/environments-details) or [apps API](deploy-api#list-apps).
*   _EnvironmentId_ (String): Unique environment identifier. Can be looked up via the [developer portal](/developerportal/deploy/environments-details) or [environments API](deploy-api#list-environments).

**Query Parameters**

*   _offset_ (Long): Number of items to offset. Default is 0.
*   _limit_ (Long): Maximum number of items in response. Default is 100.

**Example Request**

```bash
GET /api/v2/apps/543857rfds-dfsfsd12c5e24-3224d32eg/environments/cd5fc610-edb0-43c5-a374-0439a6411ace/snapshots?offset=0&limit=2
Host: deploy.mendix.com

Content-Type: application/json
Mendix-Username: richard.ford51@example.com
Mendix-ApiKey:  26587896-1cef-4483-accf-ad304e2673d6
```

#### 3.1.3 Output

An object with the following key-value pairs:

*   _snapshots_ (List): List of snapshot objects.
*   _total_ (Long): The total number of snapshots for the requested environment.
*   _offset_ (Long): The offset value of the current request.
*   _limit_ (Long): The limit value of the current request.

**Error Codes**

| HTTP Status | Error code | Description |
| --- | --- | --- |
| 400 | INVALID_PARAMETERS | Not enough parameters given. Please set project_id and environment_id parameters. |
| 400 | NOT_SUPPORTED | This endpoint can only be used with Mendix Cloud V4 |
| 403 | NO_ACCESS | The user does not have access to the backups of this environment. |
| 404 | ENVIRONMENT_NOT_FOUND | Environment not found. |
| 500 | SNAPSHOT_LISTING_FAILED | An error occurred while listing the backups. Please contact Support. |

**Example Output**

```json
{
	"limit": 5,
	"offset": 0,
	"total": 32,
	"snapshots": [
		{
			"snapshot_id": "5deda9e2-f882-4925-830c-45e73c57366e",
			"model_version": "8.12.7.11687",
			"comment": "Uploaded snapshot",
			"expires_at": "2021-08-05T18:38:41.000Z",
			"state": "completed",
			"status_message": "Completed extraction",
			"created_at": "2021-05-05T18:38:41.000Z",
			"finished_at": "2021-05-05T18:40:12.000Z",
			"updated_at": "2021-05-05T18:40:12.000Z"
		},
		{
			"snapshot_id": "bf45ed4d-3308-4fb9-876b-36453ba149bf",
			"model_version": "8.12.7.11687",
			"comment": "Automatically created nightly snapshot",
			"expires_at": "2021-05-18T01:41:27.000Z",
			"state": "completed",
			"status_message": "Completed backup creation",
			"created_at": "2021-05-04T01:41:27.000Z",
			"finished_at": "2021-05-04T01:45:47.000Z",
			"updated_at": "2021-05-04T01:45:47.000Z"
		}
	]
}
```

### 3.2 Request Creation of an Environment Snapshot

#### 3.2.1 Description

Request the creation of a snapshot of an environment. The response is a JSON object containing the `snapshot_id` attribute which identifies a snapshot. Use the `snapshot_id` in an API request to check the progress of the creation this snapshot.

```bash
HTTP Method: POST
URL: https://deploy.mendix.com/api/v2/apps/<ProjectId>/environments/<EnvironmentId>/snapshots
```

#### 3.2.2 Request

**Request Parameters**

*   _ProjectId_ (String): Unique project identifier. Can be looked up via the [developer portal](/developerportal/deploy/environments-details) or [apps API](deploy-api#list-apps).
*   _EnvironmentId_ (String): Unique environment identifier. Can be looked up via the [developer portal](/developerportal/deploy/environments-details) or [environments API](deploy-api#list-environments).

**Request Body**

A JSON object with the following attributes:

*   _comment_ (String): Optional comment for this snapshot.


**Example Request**

```bash
POST /api/v2/apps/543857rfds-dfsfsd12c5e24-3224d32eg/environments/cd5fc610-edb0-43c5-a374-0439a6411ace/snapshots
Host: deploy.mendix.com

Content-Type: application/json
Mendix-Username: richard.ford51@example.com
Mendix-ApiKey:  26587896-1cef-4483-accf-ad304e2673d6

{
     "comment" :  "My snapshot"
}
```

#### 3.2.3 Output

A JSON object with the following key-value pairs:

*   _snapshot\_id_ (String): Unique identification of the snapshot job.
*   _status\_message_ (String): Human readable status message of this job.
*   _finished\_at_ (String): ISO 8601 date and time when this job reached the end state.
*   _updated\_at_ (String): ISO 8601 date and time when this job was updated.
*   _created\_at_ (String): ISO 8601 date and time when this job was created.
*   _state_ (String): Current state of this job. It always starts with `queued` followed by `running` and eventually reaches either `failed` or `completed` end states.
*   _model\_version_ (String): Model version that was running when the snapshot was created.
*   _expires\_at_ (String): ISO 8601 date and time when this snapshot will be expired.
*   _comment_ (String): A comment describing this snapshot. Can be set by users for easier future reference.

**Error Codes**

| HTTP Status | Error code | Description |
| --- | --- | --- |
| 400 | INVALID_PARAMETERS | Not enough parameters given. Please set project_id and environment_id parameters. |
| 400 | NOT_SUPPORTED | This endpoint can only be used with Mendix Cloud V4 |
| 400 | ENVIRONMENT_BUSY | Environment is busy, please try again later or contact Support for assistance.|
| 400 | INVALID_STATE | Failed to create a backup. There is currently a maintenance action in progress. Please wait until that is finished. |
| 403 | NO_ACCESS | The user does not have access to the backups of this environment. |
| 404 | ENVIRONMENT_NOT_FOUND | Environment not found. |
| 404 | NOT_FOUND | Snapshot not found. |
| 500 | SERVICE_UNAVAILABLE | Operation failed. Please try again later or contact Support if the problem persists. |

**Example Output**

```json
{
   "status_message":null,
   "model_version":null,
   "expires_at":"2020-05-18T16:00:18.000Z",
   "finished_at":null,
   "updated_at":null,
   "snapshot_id":"51dc7872-771e-4c3e-853b-352359444db6",
   "created_at":"2020-02-18T16:00:18.000Z",
   "comment":"My snapshot",
   "state":"queued"
}
```

### 3.3 Request Status of Creation of a Snapshot

#### 3.3.1 Description

Check the current status of an ongoing snapshot creation.

```bash
HTTP Method: GET
URL: https://deploy.mendix.com/api/v2/apps/<ProjectId>/environments/<EnvironmentId>/snapshots/<SnapshotId>
```

#### 3.3.2 Request

**Request Parameters**

*   _ProjectId_ (String): Unique project identifier. Can be looked up via the [developer portal](/developerportal/deploy/environments-details) or [apps API](deploy-api#list-apps).
*   _EnvironmentId_ (String): Unique environment identifier. Can be looked up via the [developer portal](/developerportal/deploy/environments-details) or [environments API](deploy-api#list-environments).
*   _SnapshotId_ (String): Identifier of the snapshot being created.

**Example Request**

```bash
GET /api/v2/apps/543857rfds-dfsfsd12c5e24-3224d32eg/environments/cd5fc610-edb0-43c5-a374-0439a6411ace/snapshots/51dc7872-771e-4c3e-853b-352359444db6
Host: deploy.mendix.com

Content-Type: application/json
Mendix-Username: richard.ford51@example.com
Mendix-ApiKey:  26587896-1cef-4483-accf-ad304e2673d6
```

#### 3.3.3 Output

An object with the following key-value pairs:

*   _snapshot\_id_ (String): Unique identification of the snapshot job.
*   _status\_message_ (String): Human readable status message of this job.
*   _finished\_at_ (String): ISO 8601 date and time when this job reached the end state.
*   _updated\_at_ (String): ISO 8601 date and time when this job was updated.
*   _created\_at_ (String): ISO 8601 date and time when this job was created.
*   _state_ (String): Current state of this job. It always starts with `queued` followed by `running` and eventually reaches either `failed` or `completed` end states.
*   _model\_version_ (String): Model version that was running when the snapshot was created.
*   _expires\_at_ (String): ISO 8601 date and time when this snapshot will be expired.
*   _comment_ (String): A comment describing this snapshot. Can be set by users for easier future reference.

**Error Codes**

| HTTP Status | Error code | Description |
| --- | --- | --- |
| 400 | INVALID_PARAMETERS | Not enough parameters given. Please set project_id and environment_id parameters. |
| 400 | NOT_SUPPORTED | This endpoint can only be used with Mendix Cloud V4 |
| 403 | NO_ACCESS | The user does not have access to the backups of this environment. |
| 404 | ENVIRONMENT_NOT_FOUND | Environment not found. |
| 404 | NOT_FOUND | Snapshot not found. |
| 500 | SERVICE_UNAVAILABLE | Operation failed. Please try again later or contact Support if the problem persists. |
**Example Output**

```json
{
   "status_message":"Completed backup creation",
   "model_version":"1.0.0.7",
   "expires_at":"2020-05-18T16:00:18.000Z",
   "finished_at":"2020-02-18T16:00:19.000Z",
   "updated_at":"2020-02-18T16:00:19.000Z",
   "snapshot_id":"51dc7872-771e-4c3e-853b-352359444db6",
   "created_at":"2020-02-18T16:00:18.000Z",
   "comment":"Manually created snapshot",
   "state":"completed"
}
```

### 3.4 Request Creation of a Snapshot Archive

#### 3.4.1 Description

Requests the creation of an archive of a backup snapshot. The response is a JSON object containing the `archive_id` attribute which identifies an archive. use this `archive_id` in an API request to check the progress of the creation of this archive, and obtain a URL to allow you to download it.

```bash
HTTP Method: POST
URL: https://deploy.mendix.com/api/v2/apps/<ProjectId>/environments/<EnvironmentId>/snapshots/<SnapshotId>/archives
```

#### 3.4.2 Request

**Request Parameters**

*   _ProjectId_ (String): Unique project identifier. Can be looked up via the [developer portal](/developerportal/deploy/environments-details) or [apps API](deploy-api#list-apps).
*   _EnvironmentId_ (String): Unique environment identifier. Can be looked up via the [developer portal](/developerportal/deploy/environments-details) or [environments API](deploy-api#list-environments).
*   _SnapshotId_ (String): Identifier of the snapshot for which you want to create an archive.

**Query Parameters**

*   _data\_type_ (String): The type of data to retrieve. Valid types are: *database_only* and *files_and_database*. Default value is *files_and_database*.


**Example Request**

```bash
GET /api/v2/apps/543857rfds-dfsfsd12c5e24-3224d32eg/environments/cd5fc610-edb0-43c5-a374-0439a6411ace/snapshots/5f8ace23-19df-4134-bd67-c338142a6097/archives?data_type=database_only

Host: deploy.mendix.com

Content-Type: application/json
Mendix-Username: richard.ford51@example.com
Mendix-ApiKey:  26587896-1cef-4483-accf-ad304e2673d6
```

#### 3.4.3 Output

An object with the following key-value pairs:

*   _archive\_id_ (String): Unique identification of the archive job.
*   _status\_message_ (String): Human readable status message of this job.
*   _finished\_at_ (String): ISO 8601 date and time when this job reached the end state.
*   _updated\_at_ (String): ISO 8601 date and time when this job was updated.
*   _created\_at_ (String): ISO 8601 date and time when this job was created.
*   _state_ (String): Current state of this job. It always starts with `queued` followed by `running` and eventually reaches either `failed` or `completed` end states.
*   _data\_type_ (String): Type of data of the requested archive.
*   _snapshot\_id_ (String): Snapshot identifier of which this archive belongs to.
*   _url_ (String): Direct URL to the backup archive. This URL can be used with download managers.

**Error Codes**

| HTTP Status | Error code | Description |
| --- | --- | --- |
| 400 | INVALID_PARAMETERS | Not enough parameters given. Please set project_id and environment_id parameters. |
| 400 | NOT_SUPPORTED | This endpoint can only be used with Mendix Cloud V4 |
| 400 | UNSUPPORTED | Unsupported data_type |
| 403 | NO_ACCESS | The user does not have access to the backups of this environment. |
| 404 | ENVIRONMENT_NOT_FOUND | Environment not found. |
| 404 | NOT_FOUND | Snapshot not found. |
| 500 | SERVICE_UNAVAILABLE | Operation failed. Please try again later or contact Support if the problem persists.|

**Example Output**

```json
{
   "status_message":null,
   "finished_at":null,
   "updated_at":null,
   "snapshot_id":"5f8ace23-19df-4134-bd67-c338142a6097",
   "data_type":"database_only",
   "created_at":"2020-02-18T17:01:56.000Z",
   "state":"queued",
   "archive_id":"a6f519aa-a68e-4054-9341-2cfec72ea184",
   "url":null
}
```

### 3.5 Request Status of Creation of an Archive

#### 3.5.1 Description

After a request to create an archive is submitted, you can check the progress of the archive creation using the `archive_id`. The archive creation will eventually reach one of the following end states: *completed* or *failed*. When it is completed, the `url` attribute is populated with a direct link to your requested backup. This link is valid for eight hours after completion.

```bash
HTTP Method: GET
URL: https://deploy.mendix.com/api/v2/apps/<ProjectId>/environments/<EnvironmentId>/snapshots/<SnapshotId>/archives/<ArchiveId>
```

#### 3.5.2 Request

**Request Parameters**

*   _ProjectId_ (String): Unique project identifier. Can be looked up via the [developer portal](/developerportal/deploy/environments-details) or [apps API](deploy-api#list-apps).
*   _EnvironmentId_ (String): Unique environment identifier. Can be looked up via the [developer portal](/developerportal/deploy/environments-details) or [environments API](deploy-api#list-environments).
*   _SnapshotId_ (String): Identifier of the backup.
*   _ArchiveId_ (String): Identifier of the archive being created.

**Example Request**

```bash
GET /api/v2/apps/543857rfds-dfsfsd12c5e24-3224d32eg/environments/cd5fc610-edb0-43c5-a374-0439a6411ace/snapshots/5f8ace23-19df-4134-bd67-c338142a6097/archives/a6f519aa-a68e-4054-9341-2cfec72ea184

Host: deploy.mendix.com

Content-Type: application/json
Mendix-Username: richard.ford51@example.com
Mendix-ApiKey:  26587896-1cef-4483-accf-ad304e2673d6
```

#### 3.5.3 Output

An object with the following key-value pairs:

*   _archive\_id_ (String): Unique identification of the archive job.
*   _status\_message_ (String): Human readable status message of this job.
*   _finished\_at_ (String): ISO 8601 date and time when this job reached the end state.
*   _updated\_at_ (String): ISO 8601 date and time when this job was updated.
*   _created\_at_ (String): ISO 8601 date and time when this job was created.
*   _state_ (String): Current state of this job. It always starts with `queued` followed by `running` and eventually reaches either `failed` or `completed` end states.
*   _data\_type_ (String): Type of data of the requested archive.
*   _snapshot\_id_ (String): Snapshot identifier of which this archive belongs to.
*   _url_ (String): Direct URL to the backup archive. This URL can be used to download your backup archive file.

**Error Codes**

| HTTP Status | Error code | Description |
| --- | --- | --- |
| 400 | INVALID_PARAMETERS | Not enough parameters given. Please set project_id and environment_id parameters. |
| 400 | NOT_SUPPORTED | This endpoint can only be used with Mendix Cloud V4 |
| 403 | NO_ACCESS | The user does not have access to the backups of this environment. |
| 404 | ENVIRONMENT_NOT_FOUND | Environment not found. |
| 404 | NOT_FOUND | Snapshot not found. |
| 404 | NOT_FOUND | Archive not found. |


**Example Output**

```json
{
   "status_message":"Done preparing download archive",
   "finished_at":"2020-02-18T17:01:57.000Z",
   "updated_at":"2020-02-18T17:01:57.000Z",
   "snapshot_id":"5f8ace23-19df-4134-bd67-c338142a6097",
   "data_type":"database_only",
   "created_at":"2020-02-18T17:01:56.000Z",
   "state":"completed",
   "archive_id":"a6f519aa-a68e-4054-9341-2cfec72ea184",
   "url":"https://…"
}
```

### 3.6 Update an Existing Snapshot

#### 3.6.1 Description

Set a new comment for an existing snapshot. The _updated\_at_ attribute remains unchanged after this operation.

```bash
HTTP Method: PUT
URL: https://deploy.mendix.com/api/v2/apps/<ProjectId>/environments/<EnvironmentId>/snapshots/<SnapshotId>
```

#### 3.6.2 Request

**Request Parameters**

*   _ProjectId_ (String): Unique project identifier. Can be looked up via the [developer portal](/developerportal/deploy/environments-details) or [apps API](deploy-api#list-apps).
*   _EnvironmentId_ (String): Unique environment identifier. Can be looked up via the [developer portal](/developerportal/deploy/environments-details) or [environments API](deploy-api#list-environments).
*   _SnapshotId_ (String): Identifier of the backup.
*   _Comment_ (String): Optional comment for this snapshot.

**Request Body**

A JSON object with the following attributes:

*   _comment_ (String): New comment for this snapshot.

**Example Request**

```bash
PUT /api/v2/apps/543857rfds-dfsfsd12c5e24-3224d32eg/environments/cd5fc610-edb0-43c5-a374-0439a6411ace/snapshots/51dc7872-771e-4c3e-853b-352359444db6
Host: deploy.mendix.com

Content-Type: application/json
Mendix-Username: richard.ford51@example.com
Mendix-ApiKey:  26587896-1cef-4483-accf-ad304e2673d6

{
     "comment" :  "Hello"
}
```

#### 3.6.3 Output

An object with the following key-value pairs:

*   _snapshot\_id_ (String): Unique identification of the snapshot job.
*   _status\_message_ (String): Human readable status message of this job.
*   _finished\_at_ (String): ISO 8601 date and time when this job reached the end state.
*   _updated\_at_ (String): ISO 8601 date and time when this job was updated.
*   _created\_at_ (String): ISO 8601 date and time when this job was created.
*   _state_ (String): Current state of this job. It always starts with `queued` followed by `running` and eventually reaches either `failed` or `completed` end states.
*   _model\_version_ (String): Model version that was running when the snapshot was created.
*   _expires\_at_ (String): ISO 8601 date and time when this snapshot will be expired.
*   _comment_ (String): A comment describing this snapshot. Can be set by users for easier future reference.

**Error Codes**

| HTTP Status | Error code | Description |
| --- | --- | --- |
| 400 | INVALID_PARAMETERS | Not enough parameters given. Please set project_id and environment_id parameters. |
| 400 | NOT_SUPPORTED | This endpoint can only be used with Mendix Cloud V4 |
| 403 | NO_ACCESS | The user does not have access to the backups of this environment. |
| 404 | ENVIRONMENT_NOT_FOUND | Environment not found. |
| 404 | NOT_FOUND | Snapshot not found. |
| 500 | SERVICE_UNAVAILABLE | Operation failed. Please try again later or contact Support if the problem persists. |

**Example Output**

```json
{
   "status_message":"Completed backup creation",
   "model_version":"1.0.0.7",
   "expires_at":"2020-05-18T16:00:18.000Z",
   "finished_at":"2020-02-18T16:00:19.000Z",
   "updated_at":"2020-02-18T16:00:19.000Z",
   "snapshot_id":"51dc7872-771e-4c3e-853b-352359444db6",
   "created_at":"2020-02-18T16:00:18.000Z",
   "comment":"Hello",
   "state":"completed"
}
```

### 3.7 Delete an Existing Snapshot

#### 3.7.1 Description

Delete an existing snapshot.

```bash
HTTP Method: DELETE
URL: https://deploy.mendix.com/api/v2/apps/<ProjectId>/environments/<EnvironmentId>/snapshots/<SnapshotId>
```

#### 3.7.2 Request

**Request Parameters**

*   _ProjectId_ (String): Unique project identifier. Can be looked up via the [developer portal](/developerportal/deploy/environments-details) or [apps API](deploy-api#list-apps).
*   _EnvironmentId_ (String): Unique environment identifier. Can be looked up via the [developer portal](/developerportal/deploy/environments-details) or [environments API](deploy-api#list-environments).
*   _SnapshotId_ (String): Identifier of the backup.
*   _Comment_ (String): Optional comment for this snapshot.


**Example Request**

```bash
DELETE /api/v2/apps/543857rfds-dfsfsd12c5e24-3224d32eg/environments/cd5fc610-edb0-43c5-a374-0439a6411ace/snapshots/51dc7872-771e-4c3e-853b-352359444db6
Host: deploy.mendix.com

Content-Type: application/json
Mendix-Username: richard.ford51@example.com
Mendix-ApiKey:  26587896-1cef-4483-accf-ad304e2673d6
```

#### 3.7.3 Output

**Error Codes**

| HTTP Status | Error code | Description |
| --- | --- | --- |
| 400 | INVALID_PARAMETERS | Not enough parameters given. Please set project_id and environment_id parameters. |
| 400 | NOT_SUPPORTED | This endpoint can only be used with Mendix Cloud V4 |
| 403 | NO_ACCESS | The user does not have access to the backups of this environment. |
| 404 | ENVIRONMENT_NOT_FOUND | Environment not found. |
| 500 | SERVICE_UNAVAILABLE | Operation failed. Please try again later or contact Support if the problem persists. |

**Example Output**

No content is returned when a backup has been successfully removed.


### 3.8 Request a Restore of a Snapshot to an Environment

#### 3.8.1 Description

Restore a previously created backup snapshot to an environment. The environment to which the data will be restored must be stopped before using this call. The response of a successful call contains the details of the request. This call is only available for Mendix Cloud v4 applications. Please note that the `source_snapshot_id` can be a snapshot created for a different environment, similar to the "restore into" functionality in the Developer Portal.

```bash
HTTP Method: POST
URL: https://deploy.mendix.com/api/v2/apps/<ProjectId>/environments/<EnvironmentId>/restores
```

#### 3.8.2 Request

**Request Parameters**

*   _ProjectId_ (String): Unique project identifier. Can be looked up via the [developer portal](/developerportal/deploy/environments-details) or [apps API](deploy-api#list-apps).
*   _EnvironmentId_ (String): Unique environment identifier. Can be looked up via the [developer portal](/developerportal/deploy/environments-details) or [environments API](deploy-api#list-environments).

**Query Parameters**

*   _source\_snapshot\_id_ (String): Identifier of the snapshot which will be restored. This value is required and must belong to a snapshot within the same application, although it could be a different environment.
*   _db\_only_ (Boolean): Boolean flag. Set this to *true* if you are doing a database only restore operation. It defaults to *false* if not present.

    {{% alert type="warning" %}}Setting `db_only` to `true` will not restore any of your files leading to a risk that data will be missing from your app or that your app will not work as expected. Use this option with caution.{{% /alert %}}

**Example Request**

```bash
POST /api/v2/apps/b5f19af7-7453-465e-b9a1-d7556f524c1e/environments/d436e0cd-6200-4ac5-b858-849a6ddbb56a/restores?source_snapshot_id=5f8ace23-19df-4134-bd67-c338142a6097
Host: deploy.mendix.com

Content-Type: application/json
Mendix-Username: richard.ford51@example.com
Mendix-ApiKey:  26587896-1cef-4483-accf-ad304e2673d6
```

```bash
POST /api/v2/apps/b5f19af7-7453-465e-b9a1-d7556f524c1e/environments/d436e0cd-6200-4ac5-b858-849a6ddbb56a/restores?source_snapshot_id=5f8ace23-19df-4134-bd67-c338142a6097&db_only=true
Host: deploy.mendix.com

Content-Type: application/json
Mendix-Username: richard.ford51@example.com
Mendix-ApiKey:  26587896-1cef-4483-accf-ad304e2673d6
```

#### 3.8.3 Output

An object with the following key-value pairs:

*   _restore\_id_ (String): Unique identification of the restore job.
*   _status\_message_ (String): Human readable status message of this job.
*   _finished\_at_ (String): ISO 8601 date and time when this job reached the end state.
*   _updated\_at_ (String): ISO 8601 date and time when this job was updated.
*   _created\_at_ (String): ISO 8601 date and time when this job was created.
*   _state_ (String): Current state of this job. It always starts with `queued` followed by `running` and eventually reaches either `failed` or `completed` end states.
*   _source\_snapshot\_id_ (String): Identifier of the snapshot being restored.
*   _source\_environment\_id_ (String): Identifier of the environment from which the source snapshot was created.
*   _target\_environment\_id_ (String): Identifier of the target environment to which the snapshot is being restored.

**Error Codes**

| HTTP Status | Error code | Description |
| --- | --- | --- |
| 400 | INVALID_PARAMETERS | Not enough parameters given. Please set project_id and environment_id parameters. |
| 400 | NOT_SUPPORTED | This endpoint can only be used with Mendix Cloud V4 |
| 400 | NOT_FOUND | Source snapshot not found |
| 400 | INVALID_STATE | Failed to restore a backup. There is currently a maintenance action in progress. Please wait until that is finished. |
| 400 | ERROR_NOT_ALLOWED | Not allowed to restore backups. |
| 400 | ERROR_NOT_ALLOWED | Restore failed, backup is not in the right state to start restoring. |
| 400 | ERROR_NOT_ALLOWED| Please stop loft before restarting a backup. |
| 400 | ENVIRONMENT_BUSY | Environment is busy, please try again later or contact Support for assistance. |
| 403 | NO_ACCESS | The user does not have access to the backups of this environment. |
| 404 | ENVIRONMENT_NOT_FOUND | Environment not found. |
| 500 | SERVICE_UNAVAILABLE | Operation failed. Please try again later or contact Support if the problem persists. |


**Example Output**

```json
{
   "status_message":null,
   "restore_id":"11076b79-9df4-45d8-ac4b-dd79617138f5",
   "source_snapshot_id":"5f8ace23-19df-4134-bd67-c338142a6097",
   "finished_at":null,
   "updated_at":null,
   "target_environment_id":"d436e0cd-6200-4ac5-b858-849a6ddbb56a",
   "created_at":"2020-02-18T16:46:26.000Z",
   "state":"queued",
   "source_environment_id":"d436e0cd-6200-4ac5-b858-849a6ddbb56a"
}
```

### 3.9 Request Status of a Snapshot Restore

#### 3.9.1 Description

Check the status of a restore request.

```bash
HTTP Method: GET
URL: https://deploy.mendix.com/api/v2/apps/<ProjectId>/environments/<EnvironmentId>/restores/<RestoreId>
```

#### 3.9.2 Request

**Request Parameters**

*   _ProjectId_ (String): Unique project identifier. Can be looked up via the [developer portal](/developerportal/deploy/environments-details) or [apps API](deploy-api#list-apps).
*   _EnvironmentId_ (String): Unique environment identifier. Can be looked up via the [developer portal](/developerportal/deploy/environments-details) or [environments API](deploy-api#list-environments).
*   _RestoreId_ (String): Identifier of the request to restore the data.

**Example Request**

```bash
GET /api/v2/apps/b5f19af7-7453-465e-b9a1-d7556f524c1e/environments/d436e0cd-6200-4ac5-b858-849a6ddbb56a/restores/11076b79-9df4-45d8-ac4b-dd79617138f5
Host: deploy.mendix.com

Content-Type: application/json
Mendix-Username: richard.ford51@example.com
Mendix-ApiKey:  26587896-1cef-4483-accf-ad304e2673d6
```

#### 3.9.3 Output

An object with the following key-value pairs:

*   _restore\_id_ (String): Unique identification of the restore job.
*   _status\_message_ (String): Human readable status message of this job.
*   _finished\_at_ (String): ISO 8601 date and time when this job reached the end state.
*   _updated\_at_ (String): ISO 8601 date and time when this job was updated.
*   _created\_at_ (String): ISO 8601 date and time when this job was created.
*   _state_ (String): Current state of this job. It always starts with `queued` followed by `running` and eventually reaches either `failed` or `completed` end states.
*   _source\_snapshot\_id_ (String): Identifier of the snapshot being restored.
*   _source\_environment\_id_ (String): Identifier of the environment from which the source snapshot was created.
*   _target\_environment\_id_ (String): Identifier of the target environment to which the snapshot is being restored.

**Error Codes**

| HTTP Status | Error code | Description |
| --- | --- | --- |
| 400 | INVALID_PARAMETERS | Not enough parameters given. Please set project_id and environment_id parameters. |
| 400 | NOT_SUPPORTED | This endpoint can only be used with Mendix Cloud V4 |
| 400 | NOT_FOUND | Restore not found |
| 403 | NO_ACCESS | The user does not have access to the backups of this environment. |
| 404 | ENVIRONMENT_NOT_FOUND | Environment not found. |


**Example Output**

```json
{
   "status_message":"Restore completed",
   "restore_id":"11076b79-9df4-45d8-ac4b-dd79617138f5",
   "source_snapshot_id":"5f8ace23-19df-4134-bd67-c338142a6097",
   "finished_at":"2020-02-18T16:46:26.000Z",
   "updated_at":"2020-02-18T16:46:26.000Z",
   "target_environment_id":"d436e0cd-6200-4ac5-b858-849a6ddbb56a",
   "created_at":"2020-02-18T16:46:26.000Z",
   "state":"completed",
   "source_environment_id":"d436e0cd-6200-4ac5-b858-849a6ddbb56a"
}
```
