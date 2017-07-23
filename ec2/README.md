# Table of Contents

1. [Preface](README.md#markdown-header-preface)
2. [EC2 provisioning options](README.md#markdown-header-ec2-provisioning-options)

* * *

# Preface

EC2 is a Regional AWS service that can be used to provision resizable compute capacity in the Cloud. 

[*(back to the top)*](README.md#markdown-header-table-of-contents)

* * *

# EC2 provisioning options

EC2 supports the following provisioning options:

- **On Demand**. Resources are provisioned on demand and paid via a fixed-rate hourly schema. This schema is advised for short term, spiky and/or unpredictable workload applications that cannot be interrupted. Also this can be used for all the application that are being developed and tested on EC2 for the first time. AWS always charges an entire hour even if the instance is terminated before, only exception applies for spot instances (see below).
- **Reserved**. Resources are reserved and paid in advance for a certain amount of time (1 to 3 years). This option offers significant discount on the hourly charge schema and is advised for applications with steady state or predictable usage and also when payment upfront is possible.
- **Spot**. Allows to bid on the price of the compute capacity providing great saving on the hourly schema. Key terminology is as follow:
	- **Spot price**. Is the price of the instance in a specific time of the day.
	- **Bid price**. Is the price that the user wants to pay for allocating the computing capacity.
If the Spot price goes above the bid price then the instance is terminated automatically and  the partial hour not charged at all. If instead the user decides to terminate the instance on its own then the entire hour is charged. This schema is advised for urgent computing needs and for applications that have flexible start and end times.
- **Dedicated hosts**. In this case a physical EC2 server is dedicated for the application. This is advised for regulatory requirements that may not support multi tenant-virtualization (like government organisations that doesn't want to move fully in cloud). This option is good for licensing that doesn't support multi-tenancy scheme or cloud deployments. Dedicated instances can be purchased via an On-Demand or Reserved schema, in this last case 70% of discount applies.

[*(back to the top)*](README.md#markdown-header-table-of-contents)

* * *




To include

API
    DescribeImages

EBS

    API
        AttachVolume