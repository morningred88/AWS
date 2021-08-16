# S3 Fundamentals

## S3 Versioning

## Delete request for a versioning enabled bucket

## Delete marker

When versioning is enabled, a simple DELETE cannot permanently delete an object. Instead, Amazon S3 inserts a delete marker in the bucket, and that marker becomes the current version of the object with a new versioning ID.

When you try to GET an object whose current version is a delete marker, Amazon S3 behaves as though the object has been deleted (even though it has not been erased) and returns a 404 error.

To delete versioned objects **permanently**, you must use **DELETE Object versionId**. When you select a delete marker in the console, click delete. The delete maker is going to be permanent deleted. And the object is restored after the delete marker has been permanently deleted. 

### How to delete the object that is versioning enabled

If you need to delete an object that is versioning enabled, you need to select all the versions of the object, then select delete.

![Versioning_delete1](/S3_images/Versioning_delete1.png)



Then type **permanently delete** to delete the whole project

![Versioning_delete2](/S3_images/Versioning_delete2.png)

## S3 - encryption

### Encryption for entire bucket or specific project version

* Encryption can be applied just **for specific version ID or specific object if no versioning enabled**, when you upload a object. 

* We can go properties of the bucket and specify **default encryption mechanism for the bucket**. 

* The bucket default encryption can be overwritten when you upload an project to the bucket.

### Encrypting objects - 4 methods

* SSE-S3: encrypts S3 objects using keys handled & managed by AWS
* SSE-KMS: leverage AWS Key Management Service to manage encryption keys
* SSE-C: when you want to manage your own encryption keys
* Client Side Encryption

### SSE-S3

During uploading a file to the bucket, select properties> Server-side encryption settings> Specify an encryption key >Amazon S3 key (SSE-S3) 

![SSE-S3](/S3_images/SSE-S3.png)



After the file is uploaded, click on the version, you can see the version details, under Server-side encryption settings, you can see Sever-side encryption as Amazon S3 master-key (SSE-S3).

![SSE-S3-uploaded](/S3_images/SSE-S3-uploaded.png)

### SSE-KMS

During uploading a object, select properties> Server-side encryption settings> Specify an encryption key >Amazon Key Management Service Key (SSE-KMS) 

![SSE_KMS_s3_key](/S3_images/SSE_KMS_s3_key.png)

After the file is uploaded, click on the version, you can see the version details, under Server-side encryption settings, you can see Sever-side encryption as Amazon Key Management Service key (SSE-KMS).

![SSE-KMS_uploaded](/S3_images/SSE-KMS_uploaded.png)

#### AWS KMS key options

There are 3 options for selecting keys when you use SSE-KMS encryption, see the image above for the encryption setting during uploading a file.

* AWS managed key (AWS/S3)
* Choose from your AWS KMS keys
* Enter KMS key ARN

The second and third option are the same, just the key how to get the key from KMS service. You need to create a CMK (Customer master key) in KMS in advance. 

### SSE-C

SSE-C can only be done through AWS CLI, because the customer need to upload data key which cannot be done through the AWS S3 console.

## S3 bucket policies hands on

We want to add a bucket policy to block all file upload if it is not encrypted with SSE-S3.

S3 console> Select bucket versioning1218> Permissions tab> Bucket policy> Edit > Policy generator:

Service: S3

**Add two statements with conditions**

Add first statement:

![Bucket_policy1](/S3_images/Bucket_policy1.png)

Add second statement:

![Bucket_policy2](/S3_images/Bucket_policy2.png)

Then create the policy

![Bucket_policy3](/S3_images/Bucket_policy3.png)

**Below is the created bucket policy file in Json:**

Paste the policy Json file

```
{
  "Id": "Policy1638032199615",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1638031752283",
      "Action": [
        "s3:PutObject"
      ],
      "Effect": "Deny",
      "Resource": "arn:aws:s3:::versioning1218/*",
      "Condition": {
        "Null": {
          "s3:x-amz-server-side-encryption": "true"
        }
      },
      "Principal": "*"
    },
    {
      "Sid": "Stmt1638032131826",
      "Action": [
        "s3:PutObject"
      ],
      "Effect": "Deny",
      "Resource": "arn:aws:s3:::versioning1218/*",
      "Condition": {
        "StringNotEquals": {
          "s3:x-amz-server-side-encryption": "AES256"
        }
      },
      "Principal": "*"
    }
  ]
}
```

**Test the bucket Policy:**

Upload a file without encryption, the upload failed and got **Access Denied** error.

![Bucket_policy4](/S3_images/Bucket_policy4.png)
