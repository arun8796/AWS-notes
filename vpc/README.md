# Table of Contents

1. [Preface](README.md#markdown-header-preface)
2. [VPC core components](README.md#markdown-header-vpc-core-components)
    * [VPC](README.md#markdown-header-)
    * [Subnet](README.md#markdown-header-)
    * [Internet Gateway](README.md#markdown-header-)
    * [NAT Gateway](README.md#markdown-header-)
    * [Peering Connection](README.md#markdown-header-)

* * *

# Preface

Amazon VPC is a regional service that allows to provision a logically isolated section of the AWS cloud where users can launch AWS resources. VPC offers the possibility to have complete control over the virtual networking environment including:

- Selection of a specific IP address range.
- Creation of subnets.
- Configuration of route tables and network gateways.
- Configuration of NACLs for security purposes.

[*(back to the top)*](README.md#markdown-header-table-of-contents)

* * *

# VPC core components

In this paragraph the main VPC core components will be discussed, the overall Architectural Diagram is described in the image below:

![alt text](vpc-architecture.png "VPC Architecture")

## VPC

A Virtual Private Cloud is a logically isolated virtual network in the AWS Cloud that span over all the AWS Availability Zones in the selected Region (**VPC CANNOT SPAN OVER MULTIPLE REGIONS**).
When a new VPC is created three main information must be provided:

- A unique name.
- A CIDR block, which tells AWS the IP class that the VPC resources will be allocating.
- The tenancy (default/dedicated) which tells AWS if the resources in the VPC will be deployed on shared or dedicated hardware.

The CIDR block is a compact notation that is used to define a range of IP addresses in a compact form. CIDR notation is really easy to understand, any IPV4 address is made up of 4 octects (4 groups of 8 bits each, which converted into a decimal form represents all the numbers between 0 and 255). The number that follolws the IP class in a CIDR block tells how many bits, of the octect, left to tight, are fixed. 

e.g.: IP class 10.0.0.0/16 means that the initial 10.0 part is fixed, and will never change, whilst the remaining part 0.0 is variable. The resulting range of IP addresses may then be seen like :

```10.0.[0..255].[0..255]```

VPCs are subject to the following limits:

- Users can only create up to 5 VPCs per region, but this is a soft limit and can be changed via the AWS support. 
- The max size that of a CIDR block that can be used when creating a VPC cannot be bigger than /16.

In order to simplify the creation of a VPC, a wizard is provide to the user with four possible pre-set configurations:

- VPC with a Single Public Subnet Only
- VPC with Public and Private Subnets
- VPC with Public and Private Subnets and Hardware VPN Access
- VPC with a Private Subnet Only and Hardware VPN Access

When a new VPC is created the following default components are created automatically: a Route table, a NACL and a Security Group.

To simplify the deployment of AWS resources a **Default** VPC is always provided when a new AWS account is provisioned, the advise is not to delete this VPC as in order to re-create a Support ticket must be raised with AWS.


## Subnet

Subnets are segments of a VPC’s IP address range where the user can place groups of isolated resources. **Subnets are always associated to one and only one AWS Availability Zone**.
When a new subnet is cretaed the following information must be provided:

- A subnet name.
- The VPC the subnet belongs to.
- The CIDR block as a subset of the bigger CIDR block specified at the VPC level.
- The availability zone where the subnet belongs to.

When a new subnet is creted the console also shows how many IP addresses are available within that subnet. AWS always reserves 5 ip address per Subnet as follow (assumin an IP class of 10.0.0.0):

- **10.0.0.0**. Network address.
- **10.0.0.1**. VPC router.
- **10.0.0.2**. The IP address of the DNS server
- **10.0.0.3**. Reserved by AWS for future use.
- **10.0.0.255**. Network broadcast address. **AWS do not support broadcast in a VPC, therefore address stays reserved.**.

## Internet Gateway

Internet Gateway: The Amazon VPC side of a connection to the public Internet.
## NAT Gateway

A highly available, managed Network Address Translation (NAT) service for your resources in a private subnet to access the Internet.

## Peering Connection

A peering connection enables you to route traffic via private IP addresses between two peered VPCs.

[*(back to the top)*](README.md#markdown-header-table-of-contents)

* * *

