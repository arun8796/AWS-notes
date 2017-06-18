## Preface

S3 is a Regional AWS service that implements an object store in the Cloud that users can use to save their files.

Objects are uploaded in remote containers, called **Buckets**, and every bucket is provided with a name that must be unique in the AWS domain, this is because AWS uses an universal namespace, bucket naming convention rules are as follow:.

- Bucket names must be at least 3 and no more than 63 characters long.
- Bucket names must be a series of one or more labels. Adjacent labels are separated by a single period (.). Bucket names can contain lowercase letters, numbers, and hyphens. Each label must start and end with a lowercase letter or a number.
- The bucket name cannot contain underscores, end with a dash, have consecutive periods, or use dashes adjacent to periods.
- Bucket names must not be formatted as an IP address (e.g., 192.168.5.4).

A unique key, that can be constructed to mimic hierarchical structure, has to be provided when files are uploaded and retrieved, the URL format for a bucket is as follow:

https://s3-<region>.amazonaws.com/<bucket name>

Users can provision up to 100 buckets per AWS account, this limit can be changed via AWS support, and the size of a single file can vary between 0 bytes and 5TB. There is no formal limit on the number of files and/or total size that a bucket can have.

For files exceeding 100 MBytes the Multi Part upload must be used (see later for a detailed description).

## Design Considerations

Objects in S3 are made of:

- A Key.
- A Value (the bytes representing the file).
- A Version id (optional).
- Some Metadata .
- Some Sub-Resources. Which includes: Access Control Lists and Torrent (used to support the bit-torrent protocol).

S3 has been designed to follow the lexicographic order of the keys when persisting and retrieving objects, and this has an important impact on overall performance.

When the number of operation for a specific bucket is intensive AWS can decide to create partitions to allow a faster access to the information, in this case if the key used to persist the objects follow a sequential order there is an high probability that the partition, used to save/retrieve data, are always the same and this may degrade overall performances.
To avoid this issue some level of randomness should be added to the key string when persisting objects.

**Amazon S3 in all Regions provide a read-after-write consistency for PUTS of new objects and eventual consistency for overwrite PUTS and DELETES. **

This basically means that when a new object is uploaded that becomes immediately available for consumption whilst when an object is deleted, or updated, some time is necessary for the changes to become effective, this is because changes have to be propagated over the distributed backend.

A successfully Operation in S3 always returns the HTTP 200 response code.

## Storage classes

S3 support different types of storage classes, 4 at the time of writing, each provides different options:

Storage Type | Availability | Durability | Description
--- | --- | --- | ---
Standard | 99.99% Availability | 99.99999999999% Durability (11 x 9s) | Objects are always replicated for redundancy to other AWS facilities and are capable to sustain the loss of 2 facilities concurrently.
Infrequent Access | 99.99% Availability | 99.99999999999% Durability (11 x 9s) |
- Objects are always replicated for redundancy to other AWS facilities and are capable to sustain the loss of 2 facilities concurrently.
- A fee applies for object retrieval.
- Cheaper than Standard but only use if the access to information is infrequent. |



Reduced Redundant Storage
99.99% Availability
99.99% Durability
Able to sustain the loss of 1 facility.
Cheaper service but only use when is always possible to re-create the information from scratch.


Glacier
99.99999999999% Durability
4 to 5 hours object retrieval
Cheaper service but no SLA is guaranteed and object retrieval may take from 4 to 5 hours in total.

The pricing model for S3 services charges the following things: Storage, Requests, Storage Management, Data Transfer and Transfer Acceleration (through Content Delivery Network aka CDN).



## Multipart Upload

The Multipart upload API enables developers to upload large objects in parts. This API can be used to upload new large objects or make a copy of an existing object. Multipart uploading is a three-step process:
- Initiation.
- Parts upload.
- Completion.

Upon receiving the complete multipart upload request, Amazon S3 constructs the object from the uploaded parts, and developers can then access the object just as you would any other object in your bucket.
A list of all in-progress multipart uploads can be requested. The multipart API supports the stop and resume mechanism.