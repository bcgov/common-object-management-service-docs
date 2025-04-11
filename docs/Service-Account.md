## Service Account API access

Service account feature for COMS can be enabled by setting s3AccessMode enabled in configuration, it allows automated systems or services to securely interact with the COMS environment using access keys for authentication. Below are the steps for configuring and utilizing s3AccessMode.

### Credentials

For M2M API access, the storage credentials are used to authenticate API requests. These credentials consist of an Access Key ID and Access Key Secret, which are essential for machine-to-machine communication.

#### Service Account Credentials:
***Access Key ID*** - username of the object-storage user account - eg: *energy_user_1*

***Secret Access Key*** - secret or password for the object-storage user account - eg: *f6jGUSrmd9gdQg6*

#### Set Request Headers and Parameters

With s3AccessMode enabled, the M2M communication requires specific headers to authenticate and provide access correctly.

***bucket*** (*x-amz-bucket*) - Name/ID of the object-storage bucket - eg: *abcdef*

***endpoint*** (*x-amz-endpoint*) Object Storage Service Endpoint - object-storage service url - eg: *https://nrs.objectstore.gov.bc.ca/*

 The endpoint to which the request is directed.

### Security Considerations

***Store Credentials Securely***: Ensure the accesskeyid and accesskeysecret are stored securely, such as in environment variables or a secure credential vault, to prevent unauthorized access.
    
***Use HTTPS***: Make sure that all API requests are made over HTTPS to protect sensitive data during transmission.