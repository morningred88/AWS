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

![Versioning_delete1](/S3_images/Versioning_delete2.png)

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

During uploading a object, select properties> Server-side encryption settings> Specify an encryption key >Amazon S3 key (SSE-S3) 

![Versioning_delete1](/S3_images/SSE-S3.png)

