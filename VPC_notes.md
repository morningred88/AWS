# Networking - VPC

## VPC overview

### CIDR limit for a VPC

Max. CIDR per VPC is **5**

Go to your vpc > Action > Edit CIDRs, you can add up to 5 different CIDRs.

## Subnet Hands on

You can add **multiple subnets for a VPC** from the console for once.

### Auto-assign public IPv4 address

 vpc console> subnets menu> Select subnets >**Action**> **Edit subset setting**>check **Enable auto-assign public IPv4 address**> **save**

All the instance launched in the subnet will be auto assigned a public IPv4 address.

## Internet gateway & Route table

### Internet gateway

When you create a internet gateway, it is in **detached** state, you need to attach it to a VPC.

### Route table

#### Default route table

When I create a VPC with subnets, and did not associate the subnets with a specific route table, default route table is associated with all the subnets

#### Public route table

* Create a route table
* Associate it with planned public subnets
* Add route to internet gateway

Second and third step order does not matter

#### Private route table

* Create a route table 
* Associate it with planned private subnets

## Bastion hosts hands on

**Step 1: create a bastion host**

Launch  an instance in public subnet

**Step 2: create a keypair**

**Step 3: launch an EC2 instance in private subnet **

* using the created keypair

* Create a security group and associate it with the private instance
  * Inbound rule: allow SSH from security group of the bastion host.

**Step 4: Copy the pem file from the keypair to bastion host**

Instance connect to bastion instance

* Load Bestion-keypair.pem to the instance using Nano editor, the key format does not correct, change to user vim editor

* `vi Bestion-keypair.pem`

* vim command in vim window

  ```
  i -- to insert mode
  ctrl-v to paste the key file
  Press ESC key to get out of the insert mode
  :wq! to save the file
  ```

* `ls` -- to check if the key file created

* Change keypair permission: `chmod 0400 Bestion-keypair.pem`


**Step 5: SSH to private instance from Bastionhost**

`ssh ec2-user@10.0.0.93 -i Bestion-keypair.pem`

## Nat gateway hands on

* Create Nat instance in the public subset but the same AZ with the private instance in private subnet. Set Connection as public

* Update route table for private subnet, add a route to Nat gateway

  * Destination: 0.0.0.0/0
  * Target: Nat gateway

* Test

  * Use bastionhost to connect private instance,

  * `curl google.com` or `curl example.com`, you can you the HTML content. 

## NACL & Security groups hands on

### Prepare EC2 instance: Enable web server 

```
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd
sudo su
echo "hello world" > /war/www/html/index.html
```
