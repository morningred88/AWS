# Monitoring, Auditing and performance

## CloudWatch Metrics

We are required to provide all of the identifiers, **Namespace, Name, and Dimensions** to uniquely identify a single Metric

### Metric namespace

Namespace is based on AWS services, such as ELB, EC2, RDS ...

As Amazon CloudWatch is a regional service, Namespaces are unique within a Region.

![Metrics_Namespace](/Monitor_Audit_SysOps/CloudWatch_CloudTrail_images/Metrics_Namespace.png)

### Metric name

Names are unique within a Namespace, e.g., *CPUUtilization* is distinct from *DiskReadOps*

### Metric dimension

The key to distinguishing between Metrics with the same Name are Dimensions

*A dimension is a name/value pair that is part of the identity of a metric. You can assign up to 10 dimensions to a metric.*

![Metric_name_search](\Monitor_Audit_SysOps\CloudWatch_CloudTrail_images\Metric_name_search.png)

When I search a metric using name CPUUtilization,  it shows up in 10 different dimensions. 

