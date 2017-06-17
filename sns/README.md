## Preface

SNS is an AWS Regional service provided by AWS that can be used to send notifications. SNS uses **Topics** as the main message channels and a push mechanism to deliver notifications to topic subscribers.
This means that subscribers are notified in real time and don't have to check for new messages like in SQS. Supported transport protocols at the time of writing for SNS are:

- HTTP & HTTPS.
- Email & Email JSON.
- SQS.
- SMS, Mobile Push.
- Lambda.

## Subscription mechanism

When a new subscriber decides to join a topic its subscription has to be first confirmed, a Token is always included in the subscription request and is valid for **3 days**, multiple subscribers are allowed on the same topic and all of them will receive the notification upon confirmation.
Subscription confirmations works differently based on the transport protocol chosen:

- For HTTP/HTTPS notifications, Amazon SNS will first POST the confirmation message (containing a token) to the specified URL. The application monitoring the URL will have to call the **ConfirmSubscription** API with the token included in the request.
- For Email and Email-JSON notifications, Amazon SNS will send an email to the specified address containing an embedded link. The user will need to click on the embedded link to confirm the subscription request.
- For SQS notifications, Amazon SNS will enqueue a challenge message containing a token to the specified queue. The application monitoring the queue will have to call the **ConfirmSubscription** API with the token.

When a new message is sent on a topic the user can decide to customise the message format for a specific transport or let AWS manage the format. The reason why SNS support either Email and Email JSON is because the first is meant to be used by end-users whilst the second by Applications.

## Billing

With Amazon SNS, there is no minimum fee and you pay only for what you use based on the transport chosen. Cost can be seens on the AWS website. With the exception of SMS messages, Amazon SNS messages can contain up to 256 KB of text data, including XML, JSON and unformatted text.
Each 64KB chunk of published data is billed as 1 request. For example, a single API call with a 256KB payload will be billed as four requests.

## Anathomy of an SNS notification

# | Description
--- | ---
Type |
MessgaeId |
TopicArn |
Subject |
Message |
Timestamp |
SignatureVersion |
Signature |
SignatureCertURL |
UnsubscribeURL |
MessageAttributes |

## Mobile Push

SNS is capable of sendind notification to mobile devices. Currently, the following push notifications platforms are supported:

- Amazon Device Messaging (ADM)
- Apple Push Notification Service (APNS)
- Google Cloud Messaging (GCM)
- Windows Push Notification Service (WNS) for Windows 8+ and Windows Phone 8.1+
- Microsoft Push Notification Service (MPNS) for Windows Phone 7+
- Baidu Cloud Push for Android devices in China

To begin using Amazon SNS mobile push notifications, you need the following:

- Step 1: Request Credentials from Mobile Platforms: To use Amazon SNS mobile push, you must first request the necessary credentials from the mobile platforms.
- Step 2: Request Token from Mobile Platforms: You then use the returned credentials to request a token for your mobile app and device from the mobile platforms. The token you receive represents your mobile app and device.
- Step 3: Create Platform Application Object: The credentials and token are then used to create a platform application object () from Amazon SNS.
- Step 4: Create Platform Endpoint Object: The is then used to create a platform endpoint object () from Amazon SNS.
- Step 5: Publish Message to Mobile Endpoint: The is then used to publish a message to an app on a mobile device. F

** A bulk upload of existing device tokens to Amazon SNS, either via the console interface or API is possible.**

## Top APIs to remember