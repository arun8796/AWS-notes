## Preface

S3 is a Regional AWS service that implements an object store in the Cloud where users can save their files. Objects are uploaded in remote containers, called **Buckets**, and every bucket is provided with a name that must be unique in the AWS domain, this is because AWS uses an universal namespace.
Bucket names must use the following naming convention:

- The name can be between 3 and 63 characters long, and can contain only lower-case characters, numbers, periods, and dashes.
- Each label in the bucket name must start with a lowercase letter or number.
- The bucket name cannot contain underscores, end with a dash, have consecutive periods, or use dashes adjacent to periods.
- The bucket name cannot be formatted as an IP address.

A unique key, that can be constructed to mimic a hierarchical structure, has to be provided when new files are uploaded these keys are also needed for object retrieval. The bucket URL format is as follow:

```https://s3-<region>.amazonaws.com/<bucket name>```

Users can provision up to 100 buckets per AWS account, **this is a soft limit can be changed via AWS support**, and the size of a single file can vary between 0 bytes and 5TB, **this is an hard limit and cannot be changed**.
There is no formal limit on the number of files and/or total size that a bucket can have also, for files exceeding 100 MBytes, the Multi Part upload must be used (see later for a detailed description).

## Design Considerations

Objects in S3 are made of: a key, a value (the bytes representing the file), a Version id (optional), some metadata and some sub-resources (which includes: Access Control Lists and Torrent, used to support the bit-torrent protocol).

**Amazon S3 in all Regions provide a read-after-write consistency for PUTS of new objects and eventual consistency for overwrite PUTS and DELETES. ** This basically means that when a new object is uploaded that becomes immediately available for consumption whilst when an object is deleted, or updated, some time is necessary for the changes to become effective, this is because changes have to be propagated over the distributed backend.
A successfully Operation in S3 always returns the HTTP 200 response code.

S3 has been designed to follow the lexicographic order of the keys when persisting and retrieving objects, this has an important impact on overall performance. When the number of operation for a specific bucket is intensive AWS can decide to create partitions over the object keys to allow a faster access to the information, in this case if the key used to persist the objects follow a sequential order there is an high probability that the partition is always the same and this may degrade the overall performances. To avoid this issue some level of randomness should be added to the key string when persisting objects.

## Storage classes

S3 support different types of storage classes, 4 at the time of writing, each class provides different options and has different cost:

Storage Type              | Availability               | Durability                           | Description
---                       | ---                        | ---                                  | ---
Standard                  | 99.99% Availability        | 99.99999999999% Durability (11 x 9s) | Objects are always replicated for redundancy to other AWS facilities and are capable to sustain the loss of 2 facilities concurrently.
Infrequent Access         | 99.99% Availability        | 99.99999999999% Durability (11 x 9s) | **(1)** Objects are always replicated for redundancy to other AWS facilities and are capable to sustain the loss of 2 facilities concurrently.  **(2)** A fee applies for object retrieval. **(3)** Cheaper than Standard but only use if the access to information is infrequent. |
Reduced Redundant Storage | 99.99% Availability        | 99.99% Durability                    | **(1)** Able to sustain the loss of 1 facility. **(2)** Cheaper service but only use when is always possible to re-create the information from scratch.
Glacier                   | 99.99999999999% Durability | N/A                                  | **(1)** Cheaper service but no SLA is guaranteed and object retrieval may take from 4 to 5 hours in total.

The pricing model for S3 services charges the following elements: storage, number of requests, storage management, data-transfer and transfer-acceleration (through Content Delivery Network aka CDN).

## Multipart Upload

The Multipart upload API enables developers to upload large objects in parts. This API can be used to upload new large objects or make a copy of an existing object.
Multipart uploading is a three-step process: initiation, parts upload, completion.

Upon receiving the complete multipart upload request, Amazon S3 constructs the object from the uploaded parts, and developers can then access the object usual. A list of all in-progress multipart uploads can be requested. The multipart API supports the stop and resume mechanism.

## Static Website Hosting & CORS.

S3 can be used to host a static website enabling an option that can be set at bucket-level. When the Static websire option is enabled the following URL is generated:

```http://<bucket_name>.s3-website.<region>.amazonaws.com```

If a static website hosted in an S3 buckets tries to access resources into another bucket via Javascript, by default, an error is returned as S3 doesn’t allow Cross Origin Requests. There is however the possibility to enable the CORS option in the target bucket to allow a specific domain to access the bucket resources.

# Encryption

From an Encryption perspective wto aspects have to be covered in order to gurante the right level of security of the information. Those two aspects are:

- **Encryption in transit**. That is when the file moves over the network from one point to another, S3 always encrypts file in transit using the HTTPS protocol.
- **Encryption at rest**. Which refers to the file persisted in the data centre. In this case S3 supports three types of encryption mechanisms:
    - **SSE-S3** provides an integrated solution where Amazon handles key management and key protection using multiple layers of security. This option has to be choosen when is acceptable to have Amazon managing keys.
    - **SSE-C** enables you to leverage Amazon S3 to perform the encryption and decryption of your objects while retaining control of the keys used to encrypt objects. With SSE-C, you don’t need to implement or use a client-side library to perform the encryption and decryption of objects you store in Amazon S3, but you do need to manage the keys that you send to Amazon S3 to encrypt and decrypt objects. Use SSE-C if you want to maintain your own encryption keys, but don’t want to implement or leverage a client-side encryption library.
    - **SSE-KMS** enables you to use AWS Key Management Service (AWS KMS) to manage your encryption keys. Using AWS KMS to manage your keys provides several additional benefits. With AWS KMS, there are separate permissions for the use of the master key, providing an additional layer of control as well as protection against unauthorized access to your objects stored in Amazon S3. AWS KMS provides an audit trail so you can see who used your key to access which object and when, as well as view failed attempts to access data from users without permission to decrypt the data. Also, AWS KMS provides additional security controls to support customer efforts to comply with PCI-DSS, HIPAA/HITECH, and FedRAMP industry requirements.

# Versioning

Versioning is a bucket-level feature that allows to keep multiple variants of an object in the same bucket and can be used to preserve, retrieve, and restore every version of every object stored in an Amazon S3 bucket.
**Once a bucket is version-enable, it can never return to an un-versioned state, however is possible to suspend versioning on that bucket.**

The versioning state applies to all of the objects in a version-enable bucket. However is important to remember that objects stored in the bucket **before** enabling the versioning have an initial version ID of null. When versioning is enabled existing objects in the bucket do not change what instead changes is how Amazon S3 handles the objects in future requests.
When an object is deleted into a version-enabled bucket a delete marker is created and the object is not visible anymore, upon deletion of the delete marker the object would be again visible in S3.



***********************
# Cross Region Replication

Cross-region replication CRR is a bucket-level feature that enables automatic, asynchronous copying of objects across buckets in **different AWS regions**.
In the Cross Region Replication configuration users must decide which bucket has to be replicated and must provide the destination bucket where they want the objects replicated to. A subset of objects with specific key name prefixes can be selected as well.
The object replicas in the destination bucket are exact replicas of the objects in the source bucket, they have the same key names and the same metadata. Amazon S3 encrypts all data in transit across AWS regions using SSL during content replication.
In order to enable CRR following pre-requisites must be met:

- The source and destination buckets must be versioning-enabled.
- The source and destination buckets must be in different AWS regions.
- Objects can be replicated from a source bucket to only one destination bucket.
- Amazon S3 must have permission to replicate objects from that source bucket to the destination bucket on your behalf. These permissions can be granted by creating an IAM role that Amazon S3 can assume.
- If the source bucket owner also owns the object, the bucket owner has full permissions to replicate the object. If not, the source bucket owner must have permission for the Amazon S3 actions s3:GetObjectVersion and s3:GetObjectVersionACL to read the object and object ACL.
- If you are setting up cross-region replication in a cross-account scenario (where the source and destination buckets are owned by different AWS accounts), the source bucket owner must have permission to replicate objects in the destination bucket. The destination bucket owner needs to grant these permissions via a bucket policy.

Amazon S3 replicates the following things:

- Any new objects created after replication is configured. Object existing prior the replication configuration are not migrated.
- Objects created with server-side encryption using the Amazon S3-managed encryption key. The replicated copy of the object is also encrypted using server-side encryption using the Amazon S3-managed encryption key.
- Amazon S3 replicates only objects in the source bucket for which the bucket owner has permission to read objects and read ACLs.
- Any object ACL updates are replicated, although there can be some delay before Amazon S3 can bring the two in sync.
- S3 replicates object tags, if any.

For DELETE operation is also important to remember that:

- If a DELETE request is made without specifying an object version ID, Amazon S3 adds a delete marker, which cross-region replication replicates to the destination bucket.
- If a DELETE request specifies a particular object version ID to delete, Amazon S3 deletes that object version in the source bucket, but it does not replicate the deletion in the destination bucket (in other words, it does not delete the same object version from the destination bucket).

Upon restoration of a deletion marker on the source bucket is important to remember that this action doesn't get cascaded so the object stays marked as deleted in the destination bucker.