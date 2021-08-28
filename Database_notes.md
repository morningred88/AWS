# Database for SysOps

## Database backup and snapshot

Database backup is an automatic process, you can enable/disable during create the database. If you disable it, you can manually create snapshot for backup purpose. 

### Automated backups

* Daily full backup of the database (During the maintenance window)
* Transaction logs are backed-up by RDS every 5 minutes

Have ability to restore any point in time (from oldest backup to 5 minutes ago)
