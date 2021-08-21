# S3 Storage and Data Management

## S3 MFA delete

* Need MFA for **permanently delete an object version** or **suspend the versioning for a bucket**. If you just add a delete marker, you don't need MFA-Delete.
* MFA Delete **requires a versioned bucket**, because if a bucket isn't versioned, you can permanently "delete" any object (that is, you can obliterate its contents) simply by overwriting the object, which is done by creating a new object with the same key -- so requiring MFA for deletions in that case would accomplish no real purpose since it would be trivial to override or work around, whether accidentally or on purpose.
* Only **root account** can enable/disable MFA-Delete. If you have Administrator permission, you cannot enable/disable it. 
* You can **enabled/disable and use MFA-Delete using AWS CLI or API** , not in AWS console. 



## S3 default encryption

If you want to **make sure every file in the bucket is encrypted**, regardless what encryption mechanism is used, then set a default encryption for the bucket. 

### Default encryption vs bucket policies

Default encryption does not apply the same encryption to all files. When you upload a file, you can overwrite the default encryption mechanism. 

But if you wan to enforce SSE-S3 encryption for all files, you need to use bucket policy. 

## S3 replication

Including **CRR** (Cross Region Replication) and **SRR** (Same Region Replication).

* Go to origin bucket > Management > Replication rule> Create new replication rules
* You can select Create new role, AWS will automatically add new role for us.  
* Replication rule: Permanent deletion of a version will not be replicated. 
* By default, Delete marker replication is disabled. But you can enable it. 

## S3 Performance

**Improve uploading performance:**

* Multi part upload
* Transfer Accelerator

**Improve downloading performance:**

* S3 Byte-range Fetches

## S3 Select & Glacier select

* **Filter** rows and columns using **simple** SQL statement. Note that the sql statement is simple select, you cannot do aggregations.
* For complex querying, that will be serverless in S3, Amazon Athena. 

## S3 Event notifications - Hands on

S3 All object create event includes:

* Put - s3:ObjectCreated:put
* Post - s3:ObjectCreated:post
* Copy - s3:ObjectCreated:Copy
* Multipart upload completed: s3:ObjectCreated:CompletMultiPart

## S3 Glacier

### Glacier equivalent to S3 bucket

* Vault - S3 bucket
* Archive - S3 object

### Edit an object storage class:  Change from Standard to Glacier 

Select an object in a bucket > Actions> Edit storage class > choose either Glacier or Glacier deep archive

You can see the storage class for the object has changed to Glacier. You can click **Initiate restore** directly from console.

### S3 Event notification for restore object events

When you initiate a restore from S3 console, you can create a S3 event notification in the bucket.

#### Restore object event including: 

* Restore initiated - s3.ObjectRestore:Post
* Restore completed - s3.ObjectRestore:Completed
