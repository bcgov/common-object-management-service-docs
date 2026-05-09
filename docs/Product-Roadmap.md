## v1.0.0 - COMS Service Account (basic auth) access, Public Folders, Recursive Sync/Permission options, 

* [x] COMS client [Service Account access](Authentication.md#s3-service-account)
* [x] [Public folders](Permissions.md#public-access) - (share folders with unauthenticated users in BCBox)
* [x] Set files and folders as Public with S3 Bucket Policies
* [x] Recursive sync (syncing cascades down through subfolders)
* [x] Recursive folder permissions management
* [x] Bug fixes for deleted file syncing 

## v0.9.0 - CopyVersion, Sync and Folder mapping improvements

* [x] Metadata syncing optimization 
* [x] Extend permission options on file and folder invites
* [x] Bulk user permission management
* [x] Bug fixes 

## v0.8.0 - Invite Links, Pagination and S3 Public synchronization

* [x] Add paginated object search support
* [x] Add public permission tracking support from S3 Object ACL endpoints
* [x] Track S3 last modified date on versions
* [x] Add bucket last synchronized date and object last synced date
* [x] Add subfolder creation support
* [x] Improve environment variable support for recognizing truthy string representations
* [x] Add invite link deferred permission grant support (READ)
* [x] Security improvements and various bugfixes

## v0.7.0 - Synchronization

* [x] Bucket and object synchronization support
  * [x] Add retroactive backfill for existing S3 buckets and directories
  * [x] Retroactive bucket and object database population
  * [x] Implement merge-conflict logic for data collision scenarios
  * [x] Add global synchronization support for `coms-id` tags
  * [x] Add proactive `coms-id` tag annotation support for objects
  * [x] Add status probe endpoint for synchronization
* [x] Fix request timeouts from large file uploads
* [x] Remove metadata key/value length constraints
* [x] RFC 7807 error reporting compliance

## v0.6.0 - File Transit

* [x] Add new PUT endpoints for uploading files
* [x] Extend filesize limit from 50GB to 5TB
* [x] Add more consistent error responses for upload failure situations

## v0.5.0 - Filename Support

* [x] Support arbitrary filenames for S3 Objects
* [x] Add First Nations glyph support to filenames
* [x] Track and enforce `coms-id` as an S3 tag
* [x] Deprecate `coms-id` and `coms-name` S3 metadata enforcement
* [x] Add tracking support for version-specific S3 ETags

## v0.4.2 - Domain Scoped Metadata Keys

* [x] Change mandatory metadata keys to be domain scoped

## v0.4.1 - Reliability Pass

* [x] Improve architectural documentation
* [x] Refactor objection model mock pattern; improve unit test coverage
* [x] Validate bucket credentials on add and allow folder key to be optional
* [x] Performance improvements

## v0.4.0 - Multi-tenant Bucket support

* [x] Multi-bucket support
* [x] Database refactor
* [x] Permission cascade/inheritance refactor
* [x] API shape extensions

## v0.3.0 - Metadata & Tagging

* [x] Object metadata
* [x] Object tagging
* [x] Object discovery via metadata/tagging

## v0.2.0 - Version Tracking

* [x] Explicit version management
* [x] Soft-delete objects
* [x] Enhance validation layer support

## v0.1.0 - Minimum Viable Product (MVP)

### General

* [x] General Documentation
* [x] Database action auditing
* [x] Over 50% coverage in unit tests

### Authentication

* [x] Multiple authentication modes
  * [x] Unauthenticated
  * [x] Basic
  * [x] OIDC
  * [x] Full (Basic and OIDC)
* [x] Support multiple identity providers (eg: IDIR, BCeID)

### Object Operations

* [x] Upload multiple objects to storage
* [x] Expiring object download links
* [x] Object versioning and history

### Permission Management

* [x] Share objects
* [x] Search for OIDC users
* [x] Toggle object for public access
* [x] Update and manage object permissions

---