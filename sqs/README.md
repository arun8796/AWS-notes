## Preface

SQS is an AWS Regional service that provides queuing capabilities in the Cloud and is mainly being used to decouple Applications.
In SQS users define queues where producers can send messages that then gets processed by consumers (like EC2 instances, Lambada, etc.), the maximum message size is set to 256KB, **this is an hard limit** and cannot be changed.

SQS uses a poll mechanism meaning that message consumers have to cyclically check if there is a message to consume on the queue. This has also a direct impact on the billing cost as it is calculated based on the total number of requests that are sent to the SQS endpoint.
If more than 256KB need to be sent on an SQS queue, AWS advises to save these information in S3 and add a correlation identifier on the message itself to help consumers to lookup the object later on.

At the time of writing, AWS provides two main types of queues:
- Standard (available in all regions)
- FIFO (currently available in the US West (Oregon), US East (Ohio), US East (N. Virginia), and EU (Ireland) regions).

## Pricing
From a billing perspective cost is calculated based on the total number of requests also every request can be up to 64KB in size, first million requests per month is free.

**Example**: A message is 200KB in size, how many requests in total will be billed to put the message on the queue?
Solution: 200KB / 64KB = 3.125 which needs to be rounded by excess so a total of 4 requests will be billed.

## Standard queue

On a Standard queue, when a consumer picks up a message, a Timer is started, a sort of StopWatch, and the message becomes temporarily unavailable for other consumers (message is in-flight state).
If the **VisibilityTimeout** expires (Timer > VisibilityTimeout) the message becomes again visible. This means that SQS guarantees the message delivery but can't guarantee that the message is consumed only once.
Order is also undetermined as SQS uses a distributed architecture where the data is replicated across multiple AWS facilities for resilience and performance purposes.

The **VisbilityTimeout** for a single message can also be changed on the fly via an API call, **ChangeMessageVisibility**, in this case the change only affects the message and not the entire queue.

Whebn a new STandard queue is created the following attributes must be provided:

Property | Min value | Max value | Default value | Description
--- | --- | --- | --- | ---
DelaySeconds | 0 minutes | 15 minutes | 0 seconds | The time in seconds that the delivery of all messages in the queue will be delayed
MessageRetentionPeriod | 1 minute | 14 days | 4 days | The number of seconds Amazon SQS retains a message
ReceiveMessageWaitTimeSeconds | 1 second | 20 seconds | 0 seconds | Specifies the duration, in seconds, that the ReceiveMessage action call waits until a message is in the queue in order to include it in the response, as opposed to returning an empty response if a message is not yet available.
VisibilityTimeout | 0 seconds | 12 hours | 30 seconds | The length of time during which a message will be unavailable once a message is delivered from the queue

## Long vs. Short Poll

As already explained in the previous sections, messages sent to SQS are distributed over several AWS facilities and a polling mechanism is used by consumers to pick up new messages to process. Polling introduces an high number of requests being sent to the SQS endpoint with a considerable impact on the overall bill cost. This happens because AWS, by default uses a Short Polling strategy, sampling messages to delivery only from a small sub-set of the servers in the AWS facility; in this case most of the requests sent by the consumers return an empty response.
In order to minimise the overall cost a Long polling strategy may be used instead. When long polling is enabled, **ReceiveMessageWaitTimeSeconds** set to 20 seconds, AWS ill sample the messages to delivery from all the server in the AWS facility allowing to reduces the number of empty responses.

-----------------------------------


Important concepts to remind are:

SQS backend is made up of several message brokers  distributed over the AWS network for resilience and High Availability purposes. The overall infrastructure is able to scale based on demand however no action is needed from a user perspective to scale up or down.

SQS is based on a pooling mechanism, this means that consumers continuously check, via the APIs, if there are messages to process for a specific queue. By default SQS uses a short polling mechanism which basically means that only a small subset of message brokers, in the backend, gets sampled when a consumer tries to discover new messages. This behaviour causes lots of requests to be sent to AWS and lots of false responses to be generated and received by consumers, the overall volume of requests is billed as usual and this may cause cost issues. To keep the cost contained a long polling strategy can be selected via the ReceiveMessageWaitTimeSeconds property. In this case all the message brokers in the backend gets queried and messages gets delivered as soon they becomes available.

Messages in AWS SQS simple queue are not provided with a priority. If there is a need to process messages before others then multiple queues and dedicated consumers strategies have to be used.

FIFO (only available in US West and US East region). In a FIFO queue, the order is strictly guaranteed and messages get delivered exactly once, and in the order they are first delivered on the main queue. Also,  AWS has a mechanism in place to deduplicate messages sent by multiple producers over a 5 minute time window.   A limit of 300 transactions per seconds on this type of queues is in place.

FIFO queues in AWS also supports message grouping that allow to further partition messages in a FIFO queue based on a Group ID. Multiple producers can send messages to the same FIFO queue with the same group ID but by design only a single consumers is allowed to consume messages from that group ID.


SQS Fanout. Fanout is a mechanism that can be used to “glue” together AWS SNS and SQS services. In simple words an SQS queue can be subscribed to a SNS topic and received messages as soon as they are delivered on it. This is particularly useful when multiple actions needs to be performed on the same message.
