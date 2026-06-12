
The COMS API is available as a hosted service for BC Government client applications.

!!! info
    Not sure whether to use the shared hosted service, or use your own? [See here for a comparison](Hosting-Considerations.md).
    
    Self-hosting? [Get started here](Self-Hosting-COMS.md).

Some important aspects of the hosted service to consider:

### Authentication

Requests to COMS API requests can be authorized using a **User ID token** (OAuth JWT) issued by the [Pathfinder SSO "standard" realm](https://developer.gov.bc.ca/docs/default/component/css-docs/#our-service). Typically, a user would sign in to your app (website), and your app would call COMS with that user's JWT.

"Machine-to-machine" or "service account"-style access to COMS, scoped to a specified bucket, can be achieved using the [S3 Service Account flow](Authentication.md#s3-service-account).

["Global" Basic Authentication](Authentication.md#global-basic-auth) is only available if you are self-hosting COMS. It is not available on the Hosted COMS service.

Self-hosted COMS can also be configured to [disable authentication](Authentication.md#unauthenticated-mode) entirely.

COMS treats the IDIR-MFA identity provider (`azureidir`) the same as the SiteMinder-based IDIR (`idir`). In other words, whenever a user token with `idp: azureidir` is received, it is implicitly converted to `idir` and processed as such.

!!! warning
    Please see [Endpoint Notes: User](Endpoint-Notes#User) for more info on working with `azureidir` users.

### Acquiring a Bucket

An S3-compatible object storage bucket (such as AWS S3 or MinIO) must be supplied by the client. OCIO provides a low-cost [object storage service](https://ssbc-client.gov.bc.ca/services/ObjectStorage/overview.htm), and NRM clients can request a bucket through the [Optimization Team](https://apps.nrs.gov.bc.ca/int/confluence/display/OPTIMIZE/NRM+Object+Storage+Service).

Once provisioned, you can add your bucket to COMS using the [createBucket](https://coms.api.gov.bc.ca/api/v1/docs#tag/Bucket/operation/createBucket) endpoint. 

!!! info
    See here for more info on adding buckets: [Buckets](Buckets.md)

**Bucket credentials** (`Access Key ID` and `Secret Access Key`) are stored in the database as encrypted strings. Encryption is done by NodeJS's internal `crypto` library. The key for encryption is assigned to a `SERVER_PASSPHRASE` environment variable, and is only available inside the scope of the COMS app container.

### Privacy Controls

The stricter [Privacy Controls](Config.md#privacy-controls) setting is enabled in the Hosted service (requires `READ` permission on bucket or object to discover or access the file and related data). This removes the ability to search for objects that you don't have permissions for.

Only IDIR users have the ability to add buckets and search for BCeID and BC Services Card users.

### Additional features

The Hosted COMS service integrates with [BCBox](https://bcbox.nrs.gov.bc.ca/), a hosted Dropbox-style interface for sharing files.

### Environments

As part of your development workflow, ensure your application is using the correct COMS environment. 

The Hosted COMS service only accepts User Authentication tokens issued by the corresponding Pathfinder SSO "Standard" realm.

COMS has 3 environments available:
- **Development:**
	- [https://coms-dev.api.gov.bc.ca/api/v1/](https://coms.api.gov.bc.ca/api/v1/)
- **Test:**
	- [https://coms-test.api.gov.bc.ca/api/v1/](https://coms.api.gov.bc.ca/api/v1/) 
- **Production:**
	- [https://coms.api.gov.bc.ca/api/v1/](https://coms.api.gov.bc.ca/api/v1/)

!!! warning
    The **Development** and **Test** environments are unstable. Functionality may change or break at any time.
    
    Please use the **Production** environment if you require stability.
