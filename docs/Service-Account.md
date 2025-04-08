## Service Account API access

Service account feature for COMS can be enabled by setting s3AccessMode enabled in configuration, it allows automated systems or services to securely interact with the COMS environment using access keys for authentication. Below are the steps for configuring and utilizing s3AccessMode.

### Credentials

For M2M API access, the storage credentials are used to authenticate API requests. These credentials consist of an Access Key ID and Access Key Secret, which are essential for machine-to-machine communication.

#### Service Account Credentials:
***Accesskeyid:*** The unique identifier for the service account.

***Accesskeysecret:*** The associated secret key for the service account.

#### Set Request Headers and Parameters

With s3AccessMode enabled, the M2M communication requires specific headers to authenticate and provide access correctly.

***x-amz-bucket:*** The name of the bucket being accessed by the service account.

***x-amz-endpoint:*** The endpoint to which the request is directed.

### Security Considerations

***Store Credentials Securely*** Ensure the Accesskeyid and Accesskeysecret are stored securely, such as in environment variables or a secure credential vault, to prevent unauthorized access.
    
***Use HTTPS***: Make sure that all API requests are made over HTTPS to protect sensitive data during transmission.