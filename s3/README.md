## Preface

S3 is a Regional AWS service that implements an object store in the Cloud that users can use to save their files.

Objects are uploaded in remote containers, called **Buckets**, and every bucket is provided with a name that must be unique in the AWS domain, this is because AWS uses an universal namespace. Bucket naming convention rules are as follow:

- Bucket names must be at least 3 and no more than 63 characters long.
- Bucket names must be a series of one or more labels. Bucket names can contain lowercase letters, numbers, and hyphens. Each label must start and end with a lowercase letter or a number.
- The bucket name cannot contain underscores, end with a dash, have consecutive periods, or use dashes adjacent to periods.
- Bucket names must not be formatted as an IP address (e.g., 192.168.5.4).

A unique key, that can be constructed to mimic a hierarchical structure, has to be provided when files are uploaded and retrieved, the URL format for a bucket is as follow:

```https://s3-<region>.amazonaws.com/<bucket name>```

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

When the number of operation for a specific bucket is intensive AWS can decide to create partitions to allow a faster access to the information, in this case if the key used to persist the objects follow a sequential order there is an high probability that the partition, used to save/retrieve data, is always the same and this may degrade overall performances.
To avoid this issue some level of randomness should be added to the key string when persisting objects.

**Amazon S3 in all Regions provide a read-after-write consistency for PUTS of new objects and eventual consistency for overwrite PUTS and DELETES. **

This basically means that when a new object is uploaded that becomes immediately available for consumption whilst when an object is deleted, or updated, some time is necessary for the changes to become effective, this is because changes have to be propagated over the distributed backend.

A successfully Operation in S3 always returns the HTTP 200 response code.

## Storage classes

S3 support different types of storage classes, 4 at the time of writing, each provides different options:

Storage Type | Availability | Durability | Description
--- | --- | --- | ---
Standard | 99.99% Availability | 99.99999999999% Durability (11 x 9s) | Objects are always replicated for redundancy to other AWS facilities and are capable to sustain the loss of 2 facilities concurrently.
Infrequent Access | 99.99% Availability | 99.99999999999% Durability (11 x 9s) | **(1)** Objects are always replicated for redundancy to other AWS facilities and are capable to sustain the loss of 2 facilities concurrently.  **(2)** A fee applies for object retrieval. **(3)** Cheaper than Standard but only use if the access to information is infrequent. |
Reduced Redundant Storage | 99.99% Availability | 99.99% Durability | **(1)** Able to sustain the loss of 1 facility. **(2)** Cheaper service but only use when is always possible to re-create the information from scratch.
Glacier | 99.99999999999% Durability | N/A | **(1)** Cheaper service but no SLA is guaranteed and object retrieval may take from 4 to 5 hours in total.

The pricing model for S3 services charges the following things: Storage, Requests, Storage Management, Data Transfer and Transfer Acceleration (through Content Delivery Network aka CDN).

## Multipart Upload

The Multipart upload API enables developers to upload large objects in parts. This API can be used to upload new large objects or make a copy of an existing object. Multipart uploading is a three-step process:
- Initiation.
- Parts upload.
- Completion.

Upon receiving the complete multipart upload request, Amazon S3 constructs the object from the uploaded parts, and developers can then access the object just as you would any other object in your bucket.
A list of all in-progress multipart uploads can be requested. The multipart API supports the stop and resume mechanism.

## Static Website Hosting & CORS.

S3 can be used to host a static website, when this option is enabled for a bucket an access endpoint is generated with the following structure:

```http://<bucket_name>.s3-website.<region>.amazonaws.com```

If a static website, hosted in an S3 buckets, tries to access resources into another bucket via Javascript, by default, an error is returned as S3 doesn’t allow Cross Origin Requests.
There is however the possibility to enable the CORS option in the target bucket to allow a specific domain to access the bucket resources.

# Encription

# Versioning

Versioning is a bucket-level feature that allows to keep multiple variants of an object in the same bucket and can be used to preserve, retrieve, and restore every version of every object stored in an Amazon S3 bucket.
Once a bucket is version-enable, it can never return to an unversioned state, however is possible to suspend versioning on that bucket.

The versioning state applies to all of the objects in a version-enable bucket. However is important to remember that:

- Objects stored in the bucket before enabling the versioning have an initial version ID of null. When versioning is enabled existing objects in yhe bucket do not change. What changes is how Amazon S3 handles the objects in future requests.
- The bucket owner can suspend versioning to stop accruing object versions. When you suspend versioning, existing objects in your bucket do not change. What changes is how Amazon S3 handles objects in future requests.

When an object is deleted into a version-enabled bucket a delete marker is created and the object is not shown anymore, upon delition of the delete marker the vile would be again visible in S3.

# Cross Region Replication

Cross-region replication is a bucket-level feature that enables automatic, asynchronous copying of objects across buckets in different AWS regions. In the Cross Region Replication configuration user must provide information such as the destination bucket where he wants the objects replicated to. Is also possible to request Amazon S3 to replicate all or a subset of objects with specific key name prefixes.
The object replicas in the destination bucket are exact replicas of the objects in the source bucket. They have the same key names and the same metadata. Amazon S3 encrypts all data in transit across AWS regions using SSL.
Requirements for enabling cross-region replication are:

- The source and destination buckets must be versioning-enabled.
- The source and destination buckets must be in different AWS regions.
- You can replicate objects from a source bucket to only one destination bucket.
- Amazon S3 must have permission to replicate objects from that source bucket to the destination bucket on your behalf. You can grant these permissions by creating an IAM role that Amazon S3 can assume. You must grant this role permissions for Amazon S3 actions so that when Amazon S3 assumes this role, it can perform replication tasks.
- If the source bucket owner also owns the object, the bucket owner has full permissions to replicate the object. If not, the source bucket owner must have permission for the Amazon S3 actions s3:GetObjectVersion and s3:GetObjectVersionACL to read the object and object ACL. If you are setting up cross-region replication in a cross-account scenario (where the source and destination buckets are owned by different AWS accounts), the source bucket owner must have permission to replicate objects in the destination bucket. The destination bucket owner needs to grant these permissions via a bucket policy.

Amazon S3 replicates the following:

- Any new objects created after replication is configurec. Object existing prior the replication configuration are not migrated.
- Objects created with server-side encryption using the Amazon S3-managed encryption key. The replicated copy of the object is also encrypted using server-side encryption using the Amazon S3-managed encryption key.
- Amazon S3 replicates only objects in the source bucket for which the bucket owner has permission to read objects and read ACLs.
- Any object ACL updates are replicated, although there can be some delay before Amazon S3 can bring the two in sync.
- S3 replicates object tags, if any.

For DELETE operation is also important to remember that:

- If a DELETE request is made without specifying an object version ID, Amazon S3 adds a delete marker, which cross-region replication replicates to the destination bucket.
- If a DELETE request specifies a particular object version ID to delete, Amazon S3 deletes that object version in the source bucket, but it does not replicate the deletion in the destination bucket (in other words, it does not delete the same object version from the destination bucket).

Upon restoration of a deletion marker on the source bucket is important to remember that this action doesn't get cascaded so the object stays marked as deleted in the destination bucker.