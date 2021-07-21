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

Step 1: create a keypair

Step 2: launch an EC2 instance in private subnet, create a security group and allow SSH from security group of the instance for SSH connect in public subnet.

Step 3: Instance connect to bastion instance, ssh to the private instance 

* Load Bestion-keypair.pem to the instance using nano, the key format does not correct, change to user vim

* vi Bestion-keypair.pem

* vim command in vim window

  ```
  i -- to insert mode
  ctrl-v to paste the key file
  Press ESC key to get out of the insert mode
  :wq! to save the file
  ```

* ls -- to check if the key file created

* Change keypair permission: `chmod 0400 Bestion-keypair.pem`

* `ssh ec2-user@10.0.0.93 -i Bestion-keypair.pem`

