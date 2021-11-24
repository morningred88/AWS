# Advanced storage

## AWS snow family overview

* **Snowcone**: You can put Snowcone on a drone, so you can put the video data into the Snowcone. Then you can connect the Snowcone to internet, using DataSync to transfer data to S3. 
* **Snowball Edge**: You can do storage clustering, put up to 15 Snowball Edge together to increase the storage size
* **Edge location**: Anywhere does not have internet or far away from cloud
* **OpsHub**: Software you download and install in your computer/laptop. Once OpsHub is installed, it gives you graphical interface to connect your snow devices to configure and use them. 

## Storage gateway

There are 4 kinds of gateways

* **File gateway** 

  * **Store files as objects in Amazon S3, with a local cache for low-latency access to most recently used data**

  * Give you a way to expand NFS on premises by leveraging S3 in the back end. 
  * Integration between file gateway and active directory to provide authentication at file gateway level. 

* **Volume gateway: 2 types**
  * **Stored Volumes**: gateway creates AWS EBS snapshot for application server. Then the snapshot can be stored in S3. It is block storage in Amazon S3. 
  * **Cached volumes**: Low-latency access to your most recently used data.
