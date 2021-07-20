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
