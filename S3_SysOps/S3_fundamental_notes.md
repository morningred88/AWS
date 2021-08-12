# S3 Fundamentals

## S3 Versioning

## Delete request for a versioning enabled bucket

When versioning is enabled, a simple DELETE cannot permanently delete an object. Instead, Amazon S3 inserts a delete marker in the bucket, and that marker becomes the current version of the object with a new versioning ID.

When you try to GET an object whose current version is a delete marker, Amazon S3 behaves as though the object has been deleted (even though it has not been erased) and returns a 404 error.

To delete versioned objects **permanently**, you must use **DELETE Object versionId**. When you select a delete marker in the console, click delete. The delete maker is going to be permanent deleted. And the object is restored after the delete marker has been permanently deleted. 



