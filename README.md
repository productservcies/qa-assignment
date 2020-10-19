#### Description 
Following is a story we need to implement. As our QA, we would like you to provide the test scenarios needed to ensure this feature is fully tested.
- Automated testing should written in BDD style
- Language should preferably be Java 11
- If the story below is missing any ACs, please document that, ad add the relevant test scenarios
- If something is not clear, please document your assumptions, and continue based on that, or contact us for clarification
- If you need a service to interact with, please mock as needed
- We are not expecting fully working tests, but they should at least compile and be written to PROD standards       
 
##### Story
As a 10x Client
I want to be able to ascertain what messages have been published between a range of time unit
So that I can request a republication of a message produced but not received.

##### Request
```HTTP POST```

```Authorization: JWT```

URI: ```reconciliationmanager/api/v1/interval-detail-request/{client-uuid}```

<strong>Request body:</strong>

```
{
    "topicName": "<Topic>",
    "intervalFromDatetime": "<ISO8601 Formatted Datetime - Passed through from client request>",
    "intervalToDatetime": "<ISO8601 Formatted Datetime - Passed through from client request>"
}
```

##### Response
The API will return the following on success response:

```200 OK```
```
{
    "messageCount": "<total number of records in the interval, returned from Outbox Reconciliation Manager>"
}
```


The API will return the following on failure:

```500```
```
{
    "errors": [{
        "message": "A downstream service failed to respond",
        "code": "INTERNAL_SERVER_ERROR"
    }]
}
```
```404```
```
{
    "errors": [{
        "message": "Topic not supported",
        "code": "TOPIC_NOT_SUPPORTED"
    }]
}
```

```400```

```{
    "errors": [{
        "message": "Interval too large.",
        "code": "VALIDATION_FAILURE",
        "maxIntervalLength": "PT5M0S"
    }]
}
```

#### Acceptance Criteria
- AC1: When I send to a 10x publicly facing API a FROM ```intervalFromDatetime``` value and a TO ```intervalToDatetime``` value and a ```topicName``` value then 10x will publish to a Kafka client-message-reconciliation-detail-vxxx Topic a list of all message_id values for messages produced between those two boundary conditions that are associated with that given ```topicName```.
- AC2: The extent (FROM minus TO) of the date boundary that I can request can be no greater than the interval that the topic produces Outbox statistics over. If this condition is not met the request is rejected.
- AC3: Requests that are made where the FROM value is greater than the TO value are rejected.
- AC4: Requests that are made where the TO is equal to the FROM value are rejected.
- AC5: Requests that are made where the FROM value is older than 6 months are rejected.
- AC6: The interval should be left inclusive, right exclusive to guarantee no duplicate message count in the statistics.


