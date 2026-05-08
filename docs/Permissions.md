This page outlines the general design used for managing User Access Control to S3 Objects. This page targets users and developers who are planning on implementing and leveraging COMS.

## Overview

One of the core features of COMS is its focus on leveraging your specified Identity and Access Management (IAM) provider to manage access control and permissions to your resources. Secured access bucket and object resources are enforced when COMS is running in either `OIDCAUTH` or `FULLAUTH` mode. There are several notable nuances to how COMS leverages these permissions that we will discuss in further depth below.

## Access Model

COMS leverages a Discretional Access Control (DAC) model for granting access to and sharing buckets and objects. This model is used to to maximize the ability for users and clients to be able to choose at will who they wish to share their resources with. The primary benefits of the DAC model are:

1. **Simplicity** - as long as a user has a permission attached to the resource, they will be able to access the resource.
2. **Flexibility** - decentralized access control management, allowing resource owners to grant and revoke access to their objects at will without the overhead of going through a chain of command.
3. **Granularity** - the data owner is able to add or remove access permissions based on individual needs and concerns.

The key thing to take from COMS access control model is its decentralized design. The original creator of the resource will have general ownership rights to share and distribute their objects at will.

### Endpoints

There are a suite of endpoints under the `/permission` path available for users to be able to interact with the COMS permission system. These endpoints directly focus on the following goals:

- List and search for all resources that a user has explicit or implicit permissions to
- Create, update and delete permission bindings for users to bucket or object resource

Permission operation endpoints directly focus on associating users to resources with specific permissions. Existing permissions can be searched for using `GET /permission/<bucket or object>`, and standard create, read and delete operations for permissions exist to allow users to modify access control for specific resources they have management permissions over.

Any authorized user will be able to query for the current permission states to determine whether they have access to certain resources. However, only users that have the `MANAGE` permission for their associated resources will be able to modify, grant and revoke permissions to their specific resource at their discretion.

## Permission Codes

The COMS DAC model contains 5 discrete permission codes. Each of the codes represents a different set of permissions and actions that are allowed to be performed on the resource. For the most part, the permissions follow general CRUD principles and should be relatively self-explanatory.

| PermCode | Permission | Description                                                                                                                                 |
| -------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `CREATE` | Create     | Grants resource creation permission. Normally only the owner will have this permission assigned.                                            |
| `READ`   | Read       | Grants resource read permission. Ignored when in public mode for only objects.                                                              |
| `UPDATE` | Update     | Grants resource update permission. Allows user to upload a new version and/or edit metadata/tags for the object, or to edit bucket details. |
| `DELETE` | Delete     | Grants resource deletion permission. Allows user to delete objects and versions.                                                            |
| `MANAGE` | Manage     | Grants resource permission management. Allows the user to add/remove these permissions to other users.                                      |

!!! warning
    If you have `MANAGE` permissions, it is possible to delete that permission on yourself. This will lock you out of your own resources!

    COMS does not have any safeguards for accidental lockouts. You can restore access by using `PUT /permission` with [other modes of authentication](Authentication.md) if enabled, but otherwise you must contact the custodian of your COMS instance to restore your permissions on the affected resources.

### Bucket/Object Inheritance

As of COMS v0.4.0, there is now a multi-leveled permission system relating to both buckets as well as objects. While the original core concepts of the DAC model still apply, there are a few key points of note:

- Permission grants and revocations focus on binding a user to a specific resource with a specific permission code.
- Objects reside in specific buckets. As such, permissions on objects can be are now computed based on whether you have permissions in either the bucket or the object.
- You will be able to expand the response scope to also include inherited permissions, by either adding in the `bucketPerms` or `objectPerms` query parameter on the respective endpoints.

#### Examples

While the following examples are non-exhaustive, they hopefully provide a general idea of how permission transitivity applies in COMS.

Suppose Alice wishes to update object O which resides in bucket B. For this to happen, one of the following must be true:

1. Alice must have the `UPDATE` permission for Object O.
2. Alice must have the `UPDATE` permission for Bucket B. As object O resides in bucket B, the permission "cascades" to the object.

Suppose Alice wishes to manage object O which resides in bucket B. For this to happen, one of the following must be true:

1. Alice must have the `MANAGE` permission for Object O.
2. Alice must have the `MANAGE` permission for Bucket B. As object O resides in bucket B, the permission "cascades" to the object.

Suppose Alice wishes to read bucket B. For this to happen, one of the following must be true:

1. Alice must have the `READ` permission for Bucket B.
2. Alice has at least one permission binding with object O which resides in Bucket B. This will be visible by using the `objectPerms` query param. In this scenario, Alice does not have the ability to read the bucket, but they will be able to know it exists.

Suppose Alice wishes to list all buckets they have access to. For this to happen, one of the following conditions must be true:

1. Alice shall know about bucket B when they have at least one permission relation binding Alice with the bucket in question.
2. Alice shall know about bucket B when they have at least one or more objects O residing in bucket B. At least one or more permissions must bind Alice with object O. This will be visible by using the `objectPerms` query param.

#### Response Scope Expansion

There will be situations where you will want to expand the scope of your permission search to also include implicitly accessible resources. This can be done by adding either the `objectPerms` or `bucketPerms` query parameters to your API call. For example:

`GET /permission/bucket?userId=2d7f3e23-4643-47dc-b4b8-451c0844251e&objectPerms=true`

```json
[
  {
    "bucketId": "13e4e09b-5f79-48ab-985e-e4dc753a8b6a",
    "permissions": [
      {
        "id": "9fae9b19-6db3-40f2-b644-53a6c3fa87a6",
        "bucketId": "13e4e09b-5f79-48ab-985e-e4dc753a8b6a",
        "userId": "2d7f3e23-4643-47dc-b4b8-451c0844251e",
        "permCode": "CREATE",
        "createdBy": "2d7f3e23-4643-47dc-b4b8-451c0844251e",
        "createdAt": "2022-08-24T23:00:29.806Z",
        "updatedBy": null,
        "updatedAt": "2022-08-24T23:00:29.756Z"
      },
      {
        "id": "ce80040d-eb44-4170-8aea-364db8cab74a",
        "bucketId": "13e4e09b-5f79-48ab-985e-e4dc753a8b6a",
        "userId": "2d7f3e23-4643-47dc-b4b8-451c0844251e",
        "permCode": "READ",
        "createdBy": "2d7f3e23-4643-47dc-b4b8-451c0844251e",
        "createdAt": "2022-08-24T23:00:29.806Z",
        "updatedBy": null,
        "updatedAt": "2022-08-24T23:00:29.756Z"
      }
    ]
  },
  {
    "bucketId": "ce602214-8da4-48a2-a994-877e0415ea64",
    "permissions": []
  }
]
```

`GET /permission/object?userId=2d7f3e23-4643-47dc-b4b8-451c0844251e&bucketPerms=true`

```json
[
  {
    "objectId": "13e4e09b-5f79-48ab-985e-e4dc753a8b6a",
    "permissions": [
      {
        "id": "9fae9b19-6db3-40f2-b644-53a6c3fa87a6",
        "objectId": "13e4e09b-5f79-48ab-985e-e4dc753a8b6a",
        "userId": "2d7f3e23-4643-47dc-b4b8-451c0844251e",
        "permCode": "CREATE",
        "createdBy": "2d7f3e23-4643-47dc-b4b8-451c0844251e",
        "createdAt": "2022-08-24T23:00:29.806Z",
        "updatedBy": null,
        "updatedAt": "2022-08-24T23:00:29.756Z"
      },
      {
        "id": "ce80040d-eb44-4170-8aea-364db8cab74a",
        "objectId": "13e4e09b-5f79-48ab-985e-e4dc753a8b6a",
        "userId": "2d7f3e23-4643-47dc-b4b8-451c0844251e",
        "permCode": "READ",
        "createdBy": "2d7f3e23-4643-47dc-b4b8-451c0844251e",
        "createdAt": "2022-08-24T23:00:29.806Z",
        "updatedBy": null,
        "updatedAt": "2022-08-24T23:00:29.756Z"
      }
    ]
  },
  {
    "objectId": "ce602214-8da4-48a2-a994-877e0415ea64",
    "permissions": []
  }
]
```

The key thing to note with these scope expanded responses is that you gain the knowledge of a broader list of objects or buckets. However, as they are implicit inferences, they will not have any explicit permission objects in their respective arrays. For more details and clarification, reference the COMS OpenAPI specification which can be found under the `/api/v1/docs` path of your respective COMS instance.

### Mode Considerations

The above permission system will only be enforced if your instance of COMS is running in either `OIDCAUTH` or `FULLAUTH`. COMS will also require a database as it needs to have a way of persisting permission information. However, the following modes will have alternate behaviors:

- Both `NOAUTH` and `BASICAUTH` modes will completely ignore permissions as they are not in scope of permission and security enforcement. This applies whether there is a backing database or not.
- While running in `FULLAUTH` mode, if the client authenticates with a Basic authorization header, permissions are ignored as basic auth behaves as a system superuser and has "sudo" permissions to the COMS system. This applies whether there is a backing database or not.

For more specific information on COMS deployment modes and how they differ, please take a look at the COMS [Configuration guide](Config.md#authentication-modes).

## Invite Links

COMS also offers a user invite feature. Generate a time-limited, single use invitation token which can be used by an authenticated user to acquire `READ` or other permissions to a specific resource (object or bucket). Optional email-user validation may be specified to ensure the link is only used by the intended recipient. To create an invite link one must have the MANAGE permission on the resource being shared.

See [API Specification](https://coms.api.gov.bc.ca/api/v1/docs#tag/Permission/operation/createInvite)

## Public access

COMS allows you to enable public access to files, as well as buckets linked to your object storage.

This allows anyone to access files without authentication, using the COMS `GET object/:objectId` endpoint or the raw S3 object storage URL. A file set as public ignores the COMS granular permission controls.

Bucket marked as public can be accessed through the same endpoint as non-public buckets: `GET bucket/:bucketId`.

The public state of an object or bucket (which can be `true` or `false`) is included in the `GET <object|bucket>` response (eg `{ ..., public: true }`).

!!! info
    See the OpenAPI specification for details on the endpoints that toggle the public status of a file or bucket:
      - [File](https://coms.api.gov.bc.ca/api/v1/docs#tag/Object/operation/togglePublic)
      - [Bucket or Folder](https://coms.api.gov.bc.ca/api/v1/docs#tag/Bucket/operation/togglePublic)

`v1.0.0` of COMS introduced the ability to set a folder(prefix) or entire bucket as public. 

COMS uses S3 bucket policies to store the public state of files and folders. Here's an example of the Policy added by COMS to make the folder `/permits/fishing/` in bucket `abcdef` public :

```sh
# coms api REQUEST: 
# http://coms.api.gov.ba.ca/api/v1/bucket/<guid of bucket/folder>/public?public=true

# Bucket policy added to object-storage server:
{
  Action: [ 's3:GetObject', 's3:GetObjectVersion' ],
  # the prefix of the S3 path, representing the folder/bucket
  Resource: 'abcdef/permits/fishing/*', 
  Effect: 'Allow',
  Principal: '*',
  # all COMS-managed policies have a Sid prefixed with `coms::`
  Sid: 'coms::abcdef/permits/fishing/' 
}
```

### Respect for externally-set bucket policies

Pre-existing or modified files in your object storage must be synced with COMS using the COMS [synchronization process](Synchronization.md). 

Synchronization is also required to pick up any changes made to permissions ([Bucket Policies](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-policies.html) or [legacy ACLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/acl-overview.html)) applied outside of the COMS/BCBox framework, using other tools (e.g. AWS CLI. S3 Browser, etc).

- When syncing objects, the COMS `public` flag is set to `true`, if there is an external ACL or Bucket Policy that makes the object public. 
- When setting an object or bucket to public in COMS, the external policies and ACLs are preserved. COMS adds a S3 policy that enables public access, which will include a [`Sid`](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_sid.html) with `coms::` prefixed.
- When setting an object or public as private, externally-set policies are preserved. However, for single files, COMS will remove all of its ACLs.

!!! info
    ACLs are removed because they are a legacy mechanism. Although they are not deprecated as of writing, bucket policies are favoured over ACLs.

    [More info on S3 IAM policies, bucket policies, and ACLs](https://aws.amazon.com/blogs/security/iam-policies-and-bucket-policies-and-acls-oh-my-controlling-access-to-s3-resources/)

The desired end-result is that files should be identified as public if they actually are accessible without authentication.

You should always ensure you follow BC Government guidelines [before making files public](https://github.com/bcgov/bcbox/wiki/Sharing-to-the-Public).

## IDP-level permissions

Permissions may also be applied at the identity provider (IDP) level, to any bucket or object. This allows any user authenticating with the specified identity provider to receive the permission.

IDP-level permissions work in the same way as user-specific permissions. They can coexist with any bucket or object permissions and are additive. 

For instance, if a user has `DELETE` permissions on a bucket, and their identity provider has `READ` on the same bucket, the user  has both `READ` and `DELETE` permissions on that bucket.

The endpoints for reading and manipulating IDP-level permissions can be found at the path `/permission/idp`.
