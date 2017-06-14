## Preface

DynamoDB is a Regional AWS service that provides a fully managed NoSql Database in the Cloud capable of supporting both a key-value and a document based data model.
Information are always stored on SSD disks and replicated across three geographically distinct data centers; Because of these design considerations the information may require time to be propagated, therefore two different types of Consistency models for reads operation are supported:
- **Eventual Consistent model**: the consistency across all the distributed copies of the data is reached usually within a second. This option offers the best performances but also means that a read operation, after a successful write, may not return consistent information.
- **Strongly Consistent model**: no delay is introduced so the result of a read operation always reflects all the writes operation that received a successful response prior to the read. This option does not however offer best performances.
The main elements that make up DynamoDb are: **Tables** (like tables in a traditional relation database), **Items** (like rows in a traditional relation database) and **Attributes**, that can be of a primitive (String, Numbers, Booleans, etc.) or complex type, like lists or other nested objects.
``
{
   "employeeId"  : 1 ,
   "name" : "Tom",
   "Surname" "Brown",
   "dateOfBirth": "1984-02-12",
   "address" {
      "firstLine" : "...",
      "secondLine": "...",
      "postCode": "..."
   }
}
``
DynamoDB support items of up to 400KB in size with a maximum of 35 nested attributes at a time, no limit on the number of attributes that make up an item is set. These are hard limits and cannot be changed.

## Pricing

From a billing perspective two factors must be taken into consideration:
- The consistency model chosen for writing and reading items, 25 read and write capacities per month are free.
- The overall size of the data to be persisted, first 25GB per month is free.

## Primary keys and Indexes
When storing data Amazon DynamoDB divides a table into multiple partitions and distributes the data based on the value of an hash function that uses the *partition key element* (aka hash-key) of the *primary key* defined at table level. Two types of Primary key are supported:
- **Single attribute**: in this case a single attribute is chosen as Primary key and this value also acts as the *partition key element*. In this case the attribute value **cannot be duplicated within the table** (eg. employeeId).
- **Composite**: in this case two attributes are chosen, the first attribute acts as the **partition key element** whilst the second as **sort key** (aka range). In this case the partition key can have duplicates but the pair <Partition Key, Sort Key> can't. Data in partitions is also sorted by sort-key.
When designing DynamoDB tables is always important to keep the *partition* concept in mind as if the workload is heavily unbalanced, meaning disproportionately focused on one or a few partitions, the operations will not achieve the overall provisioned throughput level.
DynamoDB supports two type of indexs:
- **Local Secondary Indexes**: these can only be created when the table is first created and uses the **same partition key** defined on the Primary key of the table but allows to define a **different sort key**. These cannot be deleted or updated once table is created.
- **Global Secondary Indexes**: these can be created even after the table is created and allows to define a new partition and sort key.

## Streams

## Provisione throughput

As explained









