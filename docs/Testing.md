In the future, regression may possibly be automated, but in the meantime, here's a list of test cases to run through.

## Synchronization

- [Buckets with versioning](#buckets-with-versioning)
  - [Sync a particular bucket (versioned)](#sync-a-particular-bucket-versioned)
    - 🟢 Test case: create COMS bucket for S3 bucket with existing objects
    - 🟢 Test case: directly uploading files via S3
  - [Sync the default bucket (versioned)](#sync-the-default-bucket-versioned)
    - 🟢 Test case: upload some files to the default bucket
  - [Syncing tags (versioned)](#syncing-tags-versioned)
    - 🟢 Test case: adding tags directly via S3
    - 🟢 Test case: deleting tags directly via S3
    - 🟢 Test case: reuse `objectId` from existing `coms-id` tag
    - 🟢 Test case: deleting the `coms-id` tag
    - 🟢 Test case: syncing objects with 10 tags
  - [Sync a particular object (versioned)](#sync-a-particular-object-versioned)
    - 🟢 Test case: update existing object
    - 🟢 Test case: sync object with no new changes
    - 🟢 Test case: soft-deleting objects directly via S3
    - 🟢 Test case: hard-deleting objects directly via S3
    - 🟢 Test case: undoing a soft deletion directly via S3
- [Buckets without versioning](#buckets-without-versioning)
  - [Sync a particular bucket (unversioned)](#sync-a-particular-bucket-unversioned)
    - 🟢 Test case: create COMS bucket for S3 bucket with existing objects
    - 🟢 Test case: directly uploading files via S3
  - [Sync the default bucket (unversioned)](#sync-the-default-bucket-unversioned)
    - 🟢 Test case: upload some files to the default bucket
  - [Syncing tags (unversioned)](#syncing-tags-unversioned)
    - 🟢 Test case: adding tags directly via S3
    - 🟢 Test case: deleting tags directly via S3
    - 🟢 Test case: reuse `objectId` from existing `coms-id` tag
    - 🟢 Test case: deleting the `coms-id` tag
    - 🟢 Test case: syncing objects with 10 tags
  - [Sync a particular object (unversioned)](#sync-a-particular-object-unversioned)
    - 🟢 Test case: update existing object
    - 🟢 Test case: sync object with no new changes
    - 🟢 Test case: deleting an object

Tests should be run on all of the following types of S3 buckets:

- Buckets with versioning enabled
- Buckets with versioning disabled
- NRM object storage service (Dell ECS, but mostly S3-compliant)
  - Both versioned and unversioned buckets

### Buckets with versioning

#### Sync a particular bucket (versioned)

Sync a particular bucket so that COMS tracks all of the files in the corresponding S3 bucket.

##### 📝 Preconditions

- An existing S3 bucket is available

##### 🟢 Test case: create COMS bucket for S3 bucket with existing objects

1. Upload some files directly to an S3 bucket
2. Create a COMS bucket (i.e. `PUT /bucket`)  for said S3 bucket and save the `bucketId`
3. `GET /bucket/:bucketId/sync`
4. Call `GET /sync/status` every few seconds, until it returns `0`

- On the first call, assert that it returns `{number of files uploaded via S3}`
- Continue to call the endpoint every few seconds until returns `0`

###### Pass criteria

- `{number of files in COMS bucket} === {number of files in S3}`
- `{names of files in COMS bucket} === {names of files in S3}`
- Files marked as soft-deleted in S3 are tracked and marked as such by COMS

##### 🟢 Test case: directly uploading files via S3

1. Create a COMS bucket (i.e. `PUT /bucket`) and save the `bucketId`
2. Upload some files directly via S3
3. `GET /bucket/:bucketId/sync`
4. Call `GET /sync/status` every few seconds, until it returns `0`

- On the first call, assert that it returns `{number of files uploaded via S3}`
- Continue to call the endpoint every few seconds until returns `0`

###### Pass criteria

- `{number of files in COMS bucket} === {number of files in S3}`
- `{names of files in COMS bucket} === {names of files in S3}`

#### Sync the default bucket (versioned)

Sync the default bucket so that COMS tracks all of the files in the corresponding S3 bucket.

##### 📝 Preconditions

- COMS has been configured with a default S3 bucket

##### 🟢 Test case: upload some files to the default bucket

1. Upload some files directly via S3
2. `GET /sync`
3. Call `GET /sync/status` every few seconds, until it returns `0`

- On the first call, assert that it returns `{number of files uploaded via S3}`
- Continue to call the endpoint every few seconds until returns `0`

###### Pass criteria

- `{number of files in default COMS bucket} === {number of files in S3}`
- `{names of files in default COMS bucket} === {names of files in S3}`

#### Syncing tags (versioned)

Sync a bucket and ensure that any changes to object tags made externally are correctly synced to COMS.

##### 📝 Preconditions

- An existing S3 bucket is available and is being managed by COMS

##### 🟢 Test case: adding tags directly via S3

1. Upload file to bucket via COMS API and save the `objectId`
2. Add a tag directly via S3
3. Call `GET /sync/status` every few seconds, until it returns `0`

- On the first call, assert that it returns `{number of files uploaded via S3}`
- Continue to call the endpoint every few seconds until returns `0`

4. `GET /object/tagging?objectId={objectId}`

###### Pass criteria

- COMS tags for the object are the same as its S3 object tags

##### 🟢 Test case: deleting tags directly via S3

1. Upload file to bucket via COMS API with some tags(i.e. `PUT /object?tagset[{key}]={value}`) and save the `objectId`
2. Delete a tag directly via S3
3. Call `GET /sync/status` every few seconds, until it returns `0`

- On the first call, assert that it returns `{number of files uploaded via S3}`
- Continue to call the endpoint every few seconds until returns `0`

4. `GET /object/tagging?objectId={objectId}`

###### Pass criteria

- COMS tags for the object are the same as its S3 object tags

##### 🟢 Test case: reuse `objectId` from existing `coms-id` tag

1. Upload file to bucket via the COMS API (i.e. `PUT /object`)
2. Delete the table entry for the corresponding object in the COMS database (i.e. `DELETE FROM object WHERE id={objectId}`)
3. `GET /bucket/:bucketId/sync`
4. Call `GET /sync/status` every few seconds, until it returns `0`

- On the first call, assert that it returns `{number of files uploaded via S3}`
- Continue to call the endpoint every few seconds until returns `0`

###### Pass criteria

- The `coms-id` S3 tag is unchanged before/after deleting the object from the COMS `object` table
- `objectId` should equal the `coms-id` S3 tag

##### 🟢 Test case: deleting the `coms-id` tag

1. Upload file to bucket via COMS API and save the `objectId`
2. Delete the `coms-id` tag directly via S3
3. Call `GET /sync/status` every few seconds, until it returns `0`

- On the first call, assert that it returns `{number of files uploaded via S3}`
- Continue to call the endpoint every few seconds until returns `0`

4. `GET /object/tagging?objectId={objectId}`
5. `GET /object/:objectId/`

###### Pass criteria

- `coms-id` tag is present and has `{objectId}` as its value
- `GET /object/:objectId/` returns a HTTP 200

##### 🟢 Test case: syncing objects with 10 tags

1. Upload a file to the bucket directly via S3.
2. Add 10 tags to the file uploaded via S3.
3. `GET /bucket/:bucketId/sync`
4. Call `GET /sync/status` every few seconds, until it returns `0`

- On the first call, assert that it returns `{number of files uploaded via S3}`
- Continue to call the endpoint every few seconds until returns `0`

###### Pass criteria

- File is added to COMS with all its tags
- No `coms-id` tag is added

#### Sync a particular object (versioned)

Sync a particular object so that COMS tracks all of the changes in the corresponding S3 object.

##### 📝 Preconditions

- There is a COMS bucket that links to an existing **versioned** S3 bucket

##### 🟢 Test case: update existing object

1. Upload file via COMS API (i.e. `PUT /object`)
2. Overwrite just-uploaded file with a newer version directly via S3
3. `GET /object/:objectId/sync`
4. Call `GET /sync/status` every few seconds

- On the first call, assert that it returns `1`
- Continue to call the endpoint every few seconds until returns `0`

5. `GET /object/:objectId/`

###### Pass criteria

- `{contents of file retrieved via COMS} === {contents of file uploaded via S3}`
- `{latest version ID of file on COMS} === {latest version ID of file on S3}`

##### 🟢 Test case: sync object with no new changes

1. Upload file via COMS API (i.e. `PUT /object`)
2. `GET /object/:objectId/sync`
3. Call `GET /sync/status` every few seconds

- On the first call, assert that it returns `1`
- Continue to call the endpoint every few seconds until returns `0`

4. `GET /object/:objectId/`

###### Pass criteria

- `GET /sync/status` returns `0`
- `{contents of file retrieved via COMS} === {contents of file uploaded via S3}`
- `{latest version ID of file on COMS} === {latest version ID of file on S3}`

##### 🟢 Test case: soft-deleting objects directly via S3

1. Upload file via COMS API (i.e. `PUT /object`)
2. Soft-delete object directly via S3
3. `GET /object/:objectId/sync`
4. Call `GET /sync/status` every few seconds, until it returns `0`

- On the first call, assert that it returns `1`
- Continue to call the endpoint every few seconds until returns `0`

5. `GET /object/:objectId/`
6. `GET /object/:objectId/version`

###### Pass criteria

- `{contents of file retrieved via COMS} === {contents of file uploaded via S3}`
- `{latest S3 version ID of file on COMS} === {latest version ID of file on S3}`
  - The latest version of the COMS object is a delete marker; i.e. `isLatest === true && deleteMarker === true`

##### 🟢 Test case: hard-deleting objects directly via S3

1. Upload file via COMS API (i.e. `PUT /object`)
2. Delete latest version of object directly via S3
3. `GET /object/:objectId/sync`
4. Call `GET /sync/status` every few seconds, until it returns `0`

- On the first call, assert that it returns `1`
- Continue to call the endpoint every few seconds until returns `0`

5. `GET /object/:objectId/`

###### Pass criteria

- `GET /object/:objectId/` returns HTTP 404

##### 🟢 Test case: undoing a soft deletion directly via S3

1. Upload file via COMS API (i.e. `PUT /object`)
2. Soft-delete object via COMS API
3. Restore deleted object directly via S3 (i.e. by deleting the latest version, which should be a "delete marker" version)
4. `GET /object/:objectId/sync`
5. Call `GET /sync/status` every few seconds, until it returns `0`

- On the first call, assert that it returns `1`
- Continue to call the endpoint every few seconds until returns `0`

6. `GET /object/:objectId/`
7. `GET /object/:objectId/version`

###### Pass criteria

- `{content of file retrieved via COMS} === {contents of file version restored in S3}`
- `{latest version ID of file on COMS} === {latest version ID of file on S3}`
  - The latest version of the COMS object is not a delete marker; i.e. `isLatest === true && deleteMarker === false

### Buckets without versioning

#### Sync a particular bucket (unversioned)

Sync a particular bucket so that COMS tracks all of the files in the corresponding S3 bucket.

##### 📝 Preconditions

- An existing S3 bucket is available

##### 🟢 Test case: create COMS bucket for S3 bucket with existing objects

1. Upload some files directly to an S3 bucket
2. Create a COMS bucket (i.e. `PUT /bucket`)  for said S3 bucket and save the `bucketId`
3. `GET /bucket/:bucketId/sync`
4. Call `GET /sync/status` every few seconds, until it returns `0`

- On the first call, assert that it returns `{number of files uploaded via S3}`
- Continue to call the endpoint every few seconds until returns `0`

###### Pass criteria

- `{number of files in COMS bucket} === {number of files in S3}`
- `{names of files in COMS bucket} === {names of files in S3}`
- Files marked as soft-deleted in S3 are tracked and marked as such by COMS

##### 🟢 Test case: directly uploading files via S3

1. Create a COMS bucket (i.e. `PUT /bucket`) and save the `bucketId`
2. Upload some files directly via S3
3. `GET /bucket/:bucketId/sync`
4. Call `GET /sync/status` every few seconds, until it returns `0`

- On the first call, assert that it returns `{number of files uploaded via S3}`
- Continue to call the endpoint every few seconds until returns `0`

###### Pass criteria

- `{number of files in COMS bucket} === {number of files in S3}`
- `{names of files in COMS bucket} === {names of files in S3}`

#### Sync the default bucket (unversioned)

Sync the default bucket so that COMS tracks all of the files in the corresponding S3 bucket.

##### 📝 Preconditions

- COMS has been configured with a default S3 bucket

##### 🟢 Test case: upload some files to the default bucket

1. Upload some files directly via S3
2. `GET /sync`
3. Call `GET /sync/status` every few seconds, until it returns `0`

- On the first call, assert that it returns `{number of files uploaded via S3}`
- Continue to call the endpoint every few seconds until returns `0`

###### Pass criteria

- `{number of files in default COMS bucket} === {number of files in S3}`
- `{names of files in default COMS bucket} === {names of files in S3}`

#### Syncing tags (unversioned)

Sync a bucket and ensure that any changes to object tags made externally are correctly synced to COMS.

##### 📝 Preconditions

- An existing S3 bucket is available and is being managed by COMS

##### 🟢 Test case: adding tags directly via S3

1. Upload file to bucket via COMS API and save the `objectId`
2. Add a tag directly via S3
3. Call `GET /sync/status` every few seconds, until it returns `0`

- On the first call, assert that it returns `{number of files uploaded via S3}`
- Continue to call the endpoint every few seconds until returns `0`

4. `GET /object/tagging?objectId={objectId}`

###### Pass criteria

- COMS tags for the object are the same as its S3 object tags

##### 🟢 Test case: deleting tags directly via S3

1. Upload file to bucket via COMS API with some tags(i.e. `PUT /object?tagset[{key}]={value}`) and save the `objectId`
2. Delete a tag directly via S3
3. Call `GET /sync/status` every few seconds, until it returns `0`

- On the first call, assert that it returns `{number of files uploaded via S3}`
- Continue to call the endpoint every few seconds until returns `0`

4. `GET /object/tagging?objectId={objectId}`

###### Pass criteria

- COMS tags for the object are the same as its S3 object tags

##### 🟢 Test case: reuse `objectId` from existing `coms-id` tag

1. Upload file to bucket via the COMS API (i.e. `PUT /object`)
2. Delete the table entry for the corresponding object in the COMS database (i.e. `DELETE FROM object WHERE id={objectId}`)
3. `GET /bucket/:bucketId/sync`
4. Call `GET /sync/status` every few seconds, until it returns `0`

- On the first call, assert that it returns `{number of files uploaded via S3}`
- Continue to call the endpoint every few seconds until returns `0`

###### Pass criteria

- The `coms-id` S3 tag is unchanged before/after deleting the object from the COMS `object` table
- `objectId` should equal the `coms-id` S3 tag

##### 🟢 Test case: deleting the `coms-id` tag

1. Upload file to bucket via COMS API and save the `objectId`
2. Delete the `coms-id` tag directly via S3
3. Call `GET /sync/status` every few seconds, until it returns `0`

- On the first call, assert that it returns `{number of files uploaded via S3}`
- Continue to call the endpoint every few seconds until returns `0`

4. `GET /object/tagging?objectId={objectId}`
5. `GET /object/:objectId/`

###### Pass criteria

- `coms-id` tag is present and has `{objectId}` as its value
- `GET /object/:objectId/` returns a HTTP 200

##### 🟢 Test case: syncing objects with 10 tags

1. Upload a file to the bucket directly via S3.
2. Add 10 tags to the file uploaded via S3.
3. `GET /bucket/:bucketId/sync`
4. Call `GET /sync/status` every few seconds, until it returns `0`

- On the first call, assert that it returns `{number of files uploaded via S3}`
- Continue to call the endpoint every few seconds until returns `0`

###### Pass criteria

- File is added to COMS with all its tags
- No `coms-id` tag is added

#### Sync a particular object (unversioned)

Sync a particular object so that COMS tracks all of the changes in the corresponding S3 object.

##### 📝 Preconditions

- There is a COMS bucket that links to an existing **unversioned** S3 bucket

##### 🟢 Test case: update existing object

1. Upload file via COMS API (i.e. `PUT /object`)
2. Overwrite just-uploaded file directly via S3
3. `GET /object/:objectId/sync`
4. Call `GET /sync/status` every few seconds

- On the first call, assert that it returns `1`
- Continue to call the endpoint every few seconds until returns `0`

###### Pass criteria

- `{contents of file retrieved via COMS} === {contents of file uploaded via S3}`

##### 🟢 Test case: sync object with no new changes

1. Upload file via COMS API (i.e. `PUT /object`)
2. `GET /object/:objectId/sync`
3. Call `GET /sync/status` every few seconds

- On the first call, assert that it returns `1`
- Continue to call the endpoint every few seconds until returns `0`

4. `GET /object/:objectId/`

###### Pass criteria

- `GET /sync/status` returns `0`
- `{contents of file retrieved via COMS} === {contents of file uploaded via S3}`

##### 🟢 Test case: deleting an object

1. Upload file via COMS API (i.e. `PUT /object`)
2. Delete file directly via S3
3. `GET /object/:objectId/sync`
4. Call `GET /sync/status` every few seconds, until it returns `0`

- On the first call, assert that it returns `1`
- Continue to call the endpoint every few seconds until returns `0`

5. `GET /object/:objectId/`

###### Pass criteria

- `GET /object/:objectId/` returns HTTP 404
