# Common Object Management Service

A microservice for managing access control to files in S3-compatible Object Storage

Take advantage of more cost-effective storage solutions for your new or existing business applications. Common Object Management Service (COMS) is a secure REST API that lets you connect your application to an S3 bucket. In S3, you can store and share files, images, and documents with co-workers, partners, or the public.

## Onboarding Options

COMS is now available as a shared hosted service as well as an application that you can customise and deploy in your own infrastructure.<br />
See documentation on [Hosting Considerations](Hosting-Considerations.md).

The hosted COMS service is the engine behind [BCBox](https://bcbox.nrs.gov.bc.ca), a DropBox-like user-interface for managing files accross government.

## Feature List

- Upload, download, manage files in object storage
- Manage file versions and soft-deletes
- Enable general **public** access to files and folders
- Grant permissions on files and folders to authenticated users (eg IDIR, BCSC or BCeID)
- Share and invite users to work with your files
- Manage metadata and tags on files
- Flexible search and filter capabilities based on objects attributes and user permissions
- [OIDC Authentication](Authentication.md#oidc-authentication) using OAuth user tokens (Government SSO)
- [Service Account](Authentication.md#service-accounts) access using bucket credentials (machine-to-machine access)
- Sync COMS with files that already exist in your bucket

## Product Reference

Please follow the links in the side menu to learn more about COMS.

- Hosted COMS service URL: [https://coms.api.gov.bc.ca](https://coms.api.gov.bc.ca)
- GitHub Repository: [https://github.com/bcgov/common-object-management-service/](https://github.com/bcgov/common-object-management-service/)
- API Specification: [https://coms.api.gov.bc.ca/api/v1/docs](https://coms.api.gov.bc.ca/api/v1/docs)
- UI Integration: [BCBox](https://bcbox.nrs.gov.bc.ca)
- **[Product Roadmap](Product-Roadmap.md)**

***

### Contacts

COMS is developed by the [Common Services Team](https://bcgov.github.io/common-service-showcase/), NR Digital Services, Connected Services BC<br />
Email: <NR.CommonServiceShowcase@gov.bc.ca><br />
Community help: [the COMS/BCBox MS Teams channel](https://teams.microsoft.com/l/channel/19%3A4e700366d8aa46479a7998ffa7c86a6a%40thread.tacv2/COMS%20and%20BCBox?groupId=bef8086f-20c7-43a4-bd07-29ce764e818c&tenantId=6fdb5200-3d0d-4a8a-b036-d3685e359adc)
