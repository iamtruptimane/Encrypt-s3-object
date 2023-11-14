# Encrypting S3 Objects Using SSE-KMS
In this project we focuses on Server-Side Encryption with CMKs Stored in AWS Key Management Service (SSE-KMS). This approach gives you control of the master key that generates data keys used by S3  performing encrypt and decrypt operations. 

## Project Objectives
* Understand the benefits of SSE-KMS and when to use it
* Create customer-managed customer master keys (CMKs) in AWS Key Management Service (KMS)
* Use SSE-KMS encryption of objects at rest in S3 buckets
* Enforce that all objects in an S3 bucket are encrypted using SSE-KMS and if desired, requiring a specific CMK for the encryption

## Step 1: Logging In to the Amazon Web Services Console

## Step 2: Creating a Customer Master Key (CMK)

1. In the AWS Management Console search bar, enter KMS, and click the KMS result under Services.

2. Select Customer managed keys in the left pane of the KMS console.

3. Click Create Key, then expand Advanced Options and set the following values:
    ```
    Key type: Symmetric
    Key usage: Encrypt and decrypt
    Advanced options:
    Key Material Origin:  Leave as  KMS (default)
    Regionality: Single-Region key

    ```

4. Click Next to advance to the Add Labels page of the wizard.

5. Set the following values before clicking Next (leave the default values for other fields)
    ```
    Alias: aws-CMK-key
    Description: Customer Master Key for encrypting S3 data

    ```
6. Click Next to advance to Define Key Administrative Permissions and leave the default values.

7. Click Next to advance to Define Key Usage Permissions.

8. Click Next to preview the key policy and then click Finish when ready.  
The CMK is created.

9. Confirm the key created correctly and that the Status is Enabled.

In this step, we have learned how to manually create a Customer Master Key from the AWS console. This key can be used to (among other things) encrypt/decrypt S3 data. 

## Step 3: Encrypting S3 Data using Server-Side Encryption with KMS Managed Keys (SSE-KMS)

With your own CMK created and enabled, you are now able to use it for server-side encryption of data in S3 in the same region as the CMK. 

1. In the AWS Management Console search bar, enter S3, and click the S3 result under Services.

2. create a new bucket with unique name.

3. click on the name of the bucket.

4. Click Upload.

5. Click Add files and select a small file.

6. Expand the Properties tab and scroll until the Server-side encryption settings.

7. Check the Specify an encryption key checkbox. 

8. Check the AWS Key Management Service key (SSE-KMS) checkbox and then the Choose from your AWS KMS keys checkbox.

9. Choose the AWS KMS key you previously generated.

10. Click on Upload.

11. Click Close and then click the name of the object to open its properties panel.

You can verify the object is encrypted using SSE-KMS by checking that the Encryption field is AWS-KMS.

In this step, we have Encrypted an object in S3 using SSE-KMS. You can also configure SSE-KMS as the default encryption property for all uploaded objects by configuring bucket properties. However, that will not enforce that all objects use SSE-KMS because it can be overridden with each request. we will see how to enforce SSE-KMS encryption in the next step.

## Step 4: Enforcing S3 Encryption Using Bucket Policies

While it is useful to know how to encrypt individual objects using S3 it is often also required to enforce that all objects are encrypted in S3. Furthermore, you may require a specific encryption method and, when using SSE-KMS, a specific CMK. S3 bucket policies are the provided way for you to enforce encryption requirements of S3 buckets. You will implement a bucket policy that requires SSE-KMS to be used for all objects put in the bucket as well as the specific CMK to use for encryption.

1. In the S3 bucket console, click the Permissions tab followed by Bucket Policy to open the Bucket policy editor.

2. Paste the following bucket policy into the policy editor:
```
{
    "Version": "2012-10-17",
    "Id": "RequireSSEKMS",
    "Statement": [
        {
            "Sid": "DenyUploadIfNotSSEKMSEncrypted",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::<Your_Bucket_Name>/*",
            "Condition": {
                "StringNotEquals": {
                    "s3:x-amz-server-side-encryption": "aws:kms"
                }
            }
        }
    ]
}
```

3. Replace <Your_Bucket_Name> with the name of your bucket.

4. Click Save changes to save the policy and have it start being enforced.

5. Click the Objects tab followed by Upload.

6. Click Add files and select a small file.

7. Click Upload and observe the image does not appear in the bucket contents table.

Clicking upload without configuring any properties of the object uses the default of no encryption.

You can see the upload Failed. 

8. Retry the upload but this time use the Set properties step to configure Encryption to AWS KMS master-key using your CMK.

The upload now succeeds since the bucket policy condition is satisfied.
The policy does not require the use of your CMK however, so the default S3 KMS key in the region is also allowed. You can change the policy condition to enforce a specific CMK is used.

## Summary
In this lab step, you configured an S3 bucket policy to require SSE-KMS encryption of any new objects uploaded to the bucket. Currently, clients are required to explicitly configure the SSE-KMS properties of the object. If you prefer to automatically set the SSE-KMS properties, you can set the bucket's default encryption properties accordingly. If a client tries to override the default encryption to use a different encryption method (unencrypted or SSE-S3) the upload will fail.

