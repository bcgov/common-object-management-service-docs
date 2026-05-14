
The COMS API is available as a hosted service for BC Government client applications.

!!! info
    Not sure whether to use the shared hosted service, or use your own? [See here for a comparison](Hosting-Considerations.md).
    
    Self-hosting? [Get started here](Self-Hosting-COMS.md).

Some important aspects of the hosted service to consider:

### Authentication

- Requests to COMS API requests can be authorized using a **User ID token** (OAuth JWT) issued in the Pathfinder SSO ['Standard'](https://github.com/bcgov/sso-keycloak/wiki#standard-service) realm. Typically a user would sign-in to your app (website) and your app would call COMS with that user's JWT.

- 'Machine-to-machine' or 'service account' style access to COMS operations, scoped to a specified bucket, can be achieved using [Service Accounts](Authentication.md#s3-service-account)

- A 'global' Basic Auth is only available if you are self-hosting COMS and is not available on the Hosted COMS service.

- Self-hosted COMS can also be configured to [disable authentication](Authentication.md#unauthenticated-mode) entirely

### Acquiring a Bucket

- Object Storage buckets must be obtained by the client. Any S3 compatible bucket will work (for example: AWS S3 and Minio). OCIO provide a low-cost [object storage service](https://ssbc-client.gov.bc.ca/services/ObjectStorage/overview.htm). NRM clients can request a bucket through the [Optimization Team](https://apps.nrs.gov.bc.ca/int/confluence/display/OPTIMIZE/NRM+Object+Storage+Service).

- Once provisioned, you can add your bucket to COMS using the [createBucket](https://coms.api.gov.bc.ca/api/v1/docs#tag/Bucket/operation/createBucket) endpoint. See: [Managing Buckets](Buckets.md).

- **Bucket credentials** (`Access Key ID` and `Secret Access Key`) are stored in the database as encrypted strings. Encryption is done by NodeJS's internal `crypto` library. The key for encryption is assigned to a `SERVER_PASSPHRASE` environment variable, and is only available inside the scope of the COMS app container.

### Privacy Controls

- The stricter [Privacy Controls](Config.md#privacy-controls) setting is enabled in the Hosted service (requires `READ` permission on bucket or object to discover or access the file and related data). This removes the ability to search for objects that you don't have permissions for.

- Only IDIR users have the ability to add buckets and search for BCeID and BC Services Card users.

### Additional features

- **BCBox Integration:** Using the Hosted COMS service has the added benefit of being able to integrate your application with [BCBox](https://bcbox.nrs.gov.bc.ca/), a hosted drop-box type interface for sharing files.

### Environments

As part of your development workflow, ensure your application is using the correct COMS environment. **The Hosted COMS service only accepts User Auth tokens issued in the corresponding SSO 'Standard' realm.**

COMS environments:

- **Development:** [https://coms-dev.api.gov.bc.ca/api/v1/](https://coms.api.gov.bc.ca/api/v1/)
- **Test:** [https://coms-test.api.gov.bc.ca/api/v1/](https://coms.api.gov.bc.ca/api/v1/) 
- **Production:** [https://coms.api.gov.bc.ca/api/v1/](https://coms.api.gov.bc.ca/api/v1/)
