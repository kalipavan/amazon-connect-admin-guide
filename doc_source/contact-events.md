# Amazon Connect contact events<a name="contact-events"></a>

Amazon Connect allows you to subscribe to a near real\-time stream of contact \(voice calls, chat, and task\) events \(for example, call is queued\) in your Amazon Connect contact center\. These events include:
+ INITIATED \- A voice call, chat, or task is initiated or transferred\. 
+ CONNECTED\_TO\_SYSTEM \- The date and time the customer endpoint connected to Amazon Connect, in UTC time\. For INBOUND, this matches InitiationTimestamp\. For OUTBOUND, CALLBACK, and API, this is when the customer endpoint connected to Amazon Connect\.
**Note**  
CONNECTED\_TO\_SYSTEM event is generated for outbound calls \(API only\), Tasks, and Chats\.
+ QUEUED \- A voice call, chat, or task is queued to be assigned to an agent\.
+ CONNECTED\_TO\_AGENT \- A voice call, chat, or task is connected to an agent\.
+ DISCONNECTED \- A voice call, chat, or task is disconnected\. 

  A disconnect event is when:
  + A call, chat, or task is disconnected by an agent\.
  + A task is disconnected as a result of a flow action\.
  + A task expires\. The task is automatically disconnected if it is not completed in 7 days\. 

You can use contact events to create analytics dashboards to monitor and track contact activity, integrate into workforce management \(WFM\) solutions to better understand contact center performance, or to integrate applications that react to events \(for example, call disconnected\) in real\-time\. 

## Subscribe to Amazon Connect contact events<a name="subscribe-contact-events"></a>

Amazon Connect contact events are published using [Amazon EventBridge](http://aws.amazon.com/eventbridge/), and can be enabled in a couple of steps for your Amazon Connect instance in the Amazon EventBridge console by creating a new rule\. Although events are not ordered, they have a timestamp which enables you to consume the data\.

Events are emitted on a best effort basis\.

To subscribe to Amazon Connect contact events, go to Amazon EventBridge and create a new rule by selecting **Amazon Connect** as the service name, and **Amazon Connect contact event** as the event type\. For more information about configuring rules, see [Amazon EventBridge rules](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-rules.html) in the *Amazon EventBridge User Guide*\. 

The following image shows what this looks like in EventBridge:

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/connect/latest/adminguide/images/contact-events-eventbridge-rule.png)

You can then select a target of your choice which includes a Lambda function, SQS queue, or SNS topic\. For information about configuring targets, [Amazon EventBridge targets](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-targets.html)\. 

## Contact events data model<a name="contact-events-data-model"></a>

Contact events are generated in JSON\. For each event type, a JSON blob is sent to the target of your choice, as configured in the rule\. The following contact events are available: 
+ INITIATED \- A voice call, chat, or task is initiated or transferred\. 
+ QUEUED \- A voice call, chat, or task is queued to be assigned to an agent\.
+ CONNECTED\_TO\_AGENT \- A voice call, chat, or task is connected to an agent\.
+ CONNECTED\_TO\_SYSTEM \- This identifies that the contact has established media \(either answered by a live human voice or answering machine\)\. This event has one of the following status codes in the response schema to identify what the actual disposition was in case the contact was connected to Amazon Connect: `HUMAN_ANSWERED`, `SIT_TONE-DETECTED`, `FAX_MACHINE_DETECTED`, `VOICEMAIL_BEEP`, `VOICEMAIL_NO_BEEP`, `AMD_UNRESOLVED`, `AMD_ERROR`\. 
+ DISCONNECTED \- A voice call, chat, or task is disconnected\. For outbound calls, the dial attempt is not successful, the attempt is connected but the call is not picked up, or the attempt results in a [SIT tone](https://en.wikipedia.org/wiki/Special_information_tone)\. 

  A disconnect event is when:
  + A call, chat, or task is disconnected by an agent\.
  + A task is disconnected as a result of a flow action\.
  + A task expires\. The task is automatically disconnected if it is not completed in 7 days\. 

**Topics**
+ [Contact event](#ContactEvent)
+ [QueueInfo](#QueueInfo)
+ [AgentInfo](#AgentInfo)

### Contact event<a name="ContactEvent"></a>

The `Contact` object includes the following properties:

**EventType**  
The type of event published\.  
Type: String  
Valid values: INITIATED, QUEUED, CONNECTED\_TO\_AGENT, DISCONNECTED

**ContactId**  
The identifier for the contact\.  
Type: String  
Length: 1\-256

**InitialContactId**  
The original identifier of the contact that was transferred\.  
Type: String  
Length: 1\-256

**PreviousContactId**  
The original identifier of the contact that was transferred\.  
Type: String  
Length: 1\-256

**InstanceARN**  
Amazon Resource Name for the Amazon Connect instance in which the agent's user account is created\.  
Type: ARN

**Channel**  
The type of channel\.  
Type: `VOICE`, `CHAT`, or `TASK`

**QueueInfo**  
The queue the contact was placed in\.  
Type: `QueueInfo` object 

**AgentInfo**  
The agent the contact was assigned to\.  
Type: `AgentInfo` object 

**InitiationMethod**  
Indicates how the contact was initiated\.  
Valid values:  
+ INBOUND: The customer initiated voice \(phone\) contact with your contact center\.
+ OUTBOUND: An agent initiated voice \(phone\) contact with the customer, by using the CCP to call their number\. This initiation method calls the [StartOutboundVoiceContact](https://docs.aws.amazon.com/connect/latest/APIReference/API_StartOutboundVoiceContact.html) API\.
+ TRANSFER: The contact was transferred by an agent to another agent or to a queue, using quick connects in the CCP\. This results in a new CTR being created\.
+ CALLBACK: The customer was contacted as part of a callback flow\. For more information about the InitiationMethod in this scenario, see [About queued callbacks in metrics](about-queued-callbacks.md)\. 
+ API: The contact was initiated with Amazon Connect by API\. This could be an outbound contact you created and queued to an agent, using the [StartOutboundVoiceContact](https://docs.aws.amazon.com/connect/latest/APIReference/API_StartOutboundVoiceContact.html) API, or it could be a live chat that was initiated by the customer with your contact center, where you called the [StartChatContact](https://docs.aws.amazon.com/connect/latest/APIReference/API_StartChatContact.html) API, or it could be a tasks initiated by the customer by calling the [StartTaskContact](https://docs.aws.amazon.com/connect/latest/APIReference/API_StartTaskContact.html) API\. 
+ QUEUE\_TRANSFER: While the contact is one queue, and was then transferred into another queue using a contact flow block\.
+ DISCONNECT: When a [Set disconnect flow](set-disconnect-flow.md) block is triggered, it specifies which contact flow to run after a disconnect event\. 

  A disconnect event is when:
  + A call, chat, or task is disconnected by an agent\.
  + A task is disconnected as a result of a flow action\.
  + A task expires\. The task is automatically disconnected if it is not completed in 7 days\. 

  When the disconnect event occurs, the corresponding content flow runs\. If a new contact is created while running a disconnect flow, then the initiation method for that new contact is DISCONNECT\.

### QueueInfo<a name="QueueInfo"></a>

The `QueueInfo` object includes the following properties:

**ARN**  
The Amazon Resource Name \(ARN\) for the queue\.  
Type: String

**QueueType**  
The type of queue\.  
Type: String

### AgentInfo<a name="AgentInfo"></a>

The `AgentInfo` object includes the following properties:

**AgentARN**  
The Amazon Resource Name \(ARN\) for the agent account\.  
Type: ARN

**RoutingProfileArn**  
The Amazon Resource Name \(ARN\) for the agent's routing profile\.  
Type: String

## Contact timestamps<a name="contact-timestamps"></a>

**InitiationTimestamp**  
The date and time this contact was initiated, in UTC time\.  
Type: String \(yyyy\-MM\-dd'T'HH:mm:ss\.SSS'Z'\) 

**ConnectedToSystemTimestamp**  
The date and time the customer endpoint connected to Amazon Connect, in UTC time\.

**EnqueueTimestamp**  
The date and time the contact was added to the queue, in UTC time\.  
Type: String \(yyyy\-MM\-dd'T'HH:mm:ss\.SSS'Z'\) 

**ConnectedToAgentTimestamp**  
The date and time the contact was connected to the agent, in UTC time\.  
Type: String \(yyyy\-MM\-dd'T'HH:mm:ss\.SSS'Z'\) 

**DisconnectTimestamp**  
The date and time that the customer endpoint disconnected from Amazon Connect, in UTC time  
Type: String \(yyyy\-MM\-dd'T'HH:mm:ss\.SSS'Z'\) 

## Sample contact event for when a voice call is connected to an agent<a name="sample-contact-event"></a>

```
{
    "version": "0", 
    "id": "abcabcab-abca-abca-abca-abcabcabcabc", 
    "detail-type": "Amazon Connect Contact Event", 
    "source": "aws.connect", 
    "account": "111122223333", 
    "time": "2021-05-01T18:43:48Z",
    "region": "us-west-1", 
    "resources": [ 
        "arn:aws:...", 
        "contactArn", 
        "instanceArn"
    ],
    "detail": { 
        "eventType": "CONNECTED_TO_AGENT", 
        "contactId": "11111111-1111-1111-1111-111111111111",
        "initialContactId": "11111111-2222-3333-4444-555555555555",
        "previousContactId": "11111111-2222-3333-4444-555555555555",
        "channel": "Voice",
        "instanceARN": "arn:aws:connect:us-west-2:123456789012:instance/12345678-1234-1234-1234-123456789012",
        "initiationMethod": "INBOUND",
        "queueInfo": {
            "queueArn": "arn",       
            "queueType": "type"
        },
        "AgentInfo": {
            "AgentArn" : "arn",
            "RoutingProfileArn": ""
        }
    }
}
```

## Sample contact event for when a voice call is disconnected<a name="sample-contact-event-call-disconnected"></a>

```
{
    "version": "0",
    "id": "abcabcab-abca-abca-abca-abcabcabcabc",
    "detail-type": "Amazon Connect Contact Event",
    "source": "aws.connect",
    "account": "111122223333",
    "time": "2021-08-04T17:43:48Z",
    "region": "us-west-1",
    "resources": [
        "arn:aws:...", 
        "contactArn", 
        "instanceArn"
    ],
    "detail": {
        "eventType": "DISCONNECTED",
        "contactId": "11111111-1111-1111-1111-111111111111",
        "initialContactId": "11111111-2222-3333-4444-555555555555",
        "previousContactId": "11111111-2222-3333-4444-555555555555",
        "channel": "Voice",
        "instanceARN": "arn:aws:connect:us-west-2:123456789012:instance/12345678-1234-1234-1234-123456789012",
        "initiationMethod": "INBOUND",
        "queueInfo": {
            "queueArn": "arn",
            "queueType": "type",
            "enqueueTimestamp": "2021-08-04T17:29:04Z"
        },
        "AgentInfo": {
            "AgentArn": "arn",
            "connectedToAgentTimestamp":"2021-08-04T17:29:09Z"
        }
    },
   "initiationTimestamp":"2021-08-04T17:17:53Z",
   "connectedToSystemTimestamp":"2021-08-04T17:17:55Z",
   "disconnectTimestamp":"2021-08-04T17:18:37Z"
}
```