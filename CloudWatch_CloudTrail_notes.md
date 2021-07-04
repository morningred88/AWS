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

When I search a metric using name CPUUtilization,  it shows up in different dimensions under different namespaces. 

**Dimension for CPUUtilization in EC2: InstanceId**

Then I open EC2>Per-Instance Metrics. I can see 15 metrics, for 15 different InstanceID. 

Here the dimension name for metric CPUUtilization in namespace EC2 is InstaceID.

![Metric_name_search](\Monitor_Audit_SysOps\CloudWatch_CloudTrail_images\Dimension_InstanceId.png)

**Dimension for CPUUtilization in RDS: DBInstanceIdentifier**

You can see the same metric name under namespace RDS with dimension name DBInstanceIdentifier.

![Dimension_DBInstanceIdentifier](\Monitor_Audit_SysOps\CloudWatch_CloudTrail_images\Dimension_DBInstanceIdentifier.png)

## CloudWatch Custom metrics

CloudWatch **Unified agent** does use PutMetricData API call to push metric data to CloudWatch regularly. 
