# Disaster recovery

## AWS DataSync

* Dats Sync is not a continuous replication. It needs to be scheduled.
* It leverages DataSync agent.  

### Use cases for AWS DataSync

* **Sync data from on premise to AWS**: It can be installed on premise, using NFS or SMB protocol to connect to the file server. Then it connects to AWS DataSync via TLS, to transfer data to S3 or EFS. 
* **EFS Sync**
  * migrate to different region  
  * migrate the unencrypted EFS to encrypted DFS filesystem

## AWS Backup

* Create a backup plan to enable automatically backup
* Assign the backup plan to different resources
  *  To resources ID
  *  To specified tag

* Cross-region backups can be enabled
