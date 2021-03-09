# Common Genesys Cloud Data Actions

### Assumptions and Pre-Lab Checklist:
 * Working Genesys Cloud Org
 * Proper integration (Salesforce, Genesys Cloud, etc..) has been set up.

Custom data actions allow for Architect flows and agent scripts to get data points that are not natively available, thereby expanding their functionality.

1. Understand the business need need for a custom data action. Typically this is a customer business requirement. An example might be "if the queue doesn't have any available agents, provide an alternative treatment".
2. Find the action you want to use in the below descriptions and download the JSON file from the repository.


#### Create Callback Extended:

**Description:** Creates a callback interaction with the specified parameters. Requires phone number and queue at a minimum.

**Use Case:** Can be used by a flow to create an appointment for a callback or if the queue is busy. Could allow for a customer to choose when the callback would occur. Supports various routing settings such as priority, preferred agent, language, and skill.

**Known Issues:** None


#### External Contact Query:

**Description:** Gets details about an external contact in Genesys Cloud. Using a string to search, this will return the first result found that matches the searchAttribute.

**Use Case:** Useful if you need to do TTS readback of caller details or caller verification and all persons calling are likely to be setup as external contacts. Would be helpful if the organization has a sync process for external contact data in place or uses Genesys Cloud for these functions natively.

**Known Issues:** None


#### Get On Queue Agent Counts:

**Description:** For a given queue, this action will return the number of agents on queue by routing status. Replaces previous implementations due to API changes! See https://developer.mypurecloud.com/forum/t/removing-totals-counts-from-the-queue-users-get-query/6106

**Use Case:** Typically referred to as a staff check, the flow could offer an alternative treatment when agents are not on-queue but expected to be. Unplanned emergency routing is a typical scenario.

The data action outputs 2 parallel arrays. It outputs one array `counts` that includes the count of on queue agent by routing status. The second array is `statuses` which is the list of routing statuses so you can grab the appropriate count that you are looking for.

*Example usage*: output variables in the example are Flow.QueueCounts for the counts output, and Flow.StatusList for the statuses output

*Check if there are any agents on queue*: `sum(Flow.QueueCounts) > 0`

*Get number of idle agents*: `If(FindFirst(Flow.StatusList, "IDLE") >= 0, GetAt(Flow.QueueCounts, FindFirst(Flow.StatusList, "IDLE")), 0)`

*Get number of interacting agents*: `If(FindFirst(Flow.StatusList, "INTERACTING") >= 0, GetAt(Flow.QueueCounts, FindFirst(Flow.StatusList, "INTERACTING")), 0)`

*Get number of not responding agents*: `If(FindFirst(Flow.StatusList, "INTERACTING") >= 0, GetAt(Flow.QueueCounts, FindFirst(Flow.StatusList, "NOT_RESPONDING")), 0)`

**Known Issues:** None


#### Get Emergency Group Details:

**Description:** This action will return the name, status, and first 21 architect flows that are affected by a particular emergency group.

**Use Case:** During an in-queue flow, sometimes customers wish to check the status of an emergency group and disconnect callers if an emergency has been activated since they were placed in queue.

**Known Issues:** None


#### Get User ID by Email:

**Description:** This action will return the Genesys cloud userId of the user identified by the input email address.

**Use Case:** Should an external system return an email address belonging to a Genesys cloud user, this action will allow the conversion of the email address to a user ID which can be used in flows or subsequent data actions. A good example might be Salesforce returns the email address of the user assigned to a specific customer. This email address can then be converted to the Genesys cloud userID and used in call routing to deliver the call to the user.

**Known Issues:** None


#### Get Current Active Conversation ID by User ID:

**Description:** This action returns the most recently started ConversationId of a given user that has not yet ended.

**Use Case:** This was used to check to see if a given user was on a call before sending a DID call to them. This action returns a value for both ACD and non-ACD calls. If no such call currently exists then the value of "Not Found" is returned.

**Known Issues:** This is currently written for voice only interactions but could be adopted for any media type.
This action is written against the analytics API and therefore requires the appropriate permissions for such a query.
This action requires an interval be provided as an input. I recommend the following expression for voice calls.
```ToString(AddDays(GetCurrentDateTimeUtc(),-2))+"/"+ToString(AddHours(GetCurrentDateTimeUtc(),1))```


#### Get Estimated Wait Time With Default:

**Description:** This action will return the current estimated wait time for a given queue and media type. Should the API not return a value, zero will be used as the default.

**Use Case:** This functionality is often used by In-Queue flows to offer different treatments based on wait times for the given queue. Example might be if EWT is >300 seconds (5mins) then immediately offer the caller the option to leave a callback.

**Known Issues:** None


#### Get Generic Attribute:

**Description:** This action uses a data table to query custom settings for an organization which can be used in logic statements. Settings are global to organization.

**Use Case:** This functionality can be used to create IVR based administration for call flows or any number of custom settings for an organization. The usefulness of the settings is dependent on the customer requirements. The customer this was created for wanted to be able to modify IVR behavior using DTMF inputs from call center managers. This allowed the designers to limit the control of the contact center management to one of several predefined choices. Requires a data table with the following format;

| AttributeName (key)  |  AttributeValue  |
| -------------------- | ---------------- |
| Attribute1           |  Value1          |
| Attribute2           |  Value2          |


**Known Issues:** Often used in conjunction with the Update Generic Attribute action.
May benefit from translationMapDefaults if attribute name is not present in data table.


#### Update Generic Attribute:

**Description:** This action uses a data table to modify custom settings for an organization which can be queried and used in logic statements. Settings are global to organization.

**Use Case:** This functionality can be used to create IVR based administration for call flows or any number of custom settings for an organization. The usefulness of the settings is dependent on the customer requirements. The customer this was created for wanted to be able to modify IVR behavior using DTMF inputs from call center managers. This allowed the designers to limit the control of the contact center management to one of several predefined choices. Requires a data table with the following format;

| AttributeName (key)  |  AttributeValue  |
| -------------------- | ---------------- |
| Attribute1           |  Value1          |
| Attribute2           |  Value2          |


**Known Issues:** Often used in conjunction with the Get Generic Attribute action.
AttributeName must already exist in data table for action to update, will not create new rows in data table.
May benefit from translationMapDefaults if attribute name is not present in data table.


#### Get NWS/VIC/ACT/TAS Time Offset:

**Description:** This action will return the current GMT offset for the "Australia/Sydney" timezone.

**Use Case:** Need use case for this.

**Known Issues:** None


#### Get Interactions In Queue:

**Description:** For a given queue returns number of interactions waiting or interacting with a given media type.

**Use Case:** This can be used for flows where alternative treatments are offered once the queue size is either above a certain number or below a certain number of interactions.

**Known Issues:** Uses analytics API and therefore must have appropriate permissions.


#### Get User Out of Office Status:

**Description:** For a given user returns a true/false value indicating that their out of office is currently active.

**Use Case:** This can be used for DID calls or where if a user is out of the office the caller is provided an alternative treatment. Useful when directed routing is being used by a particular organization.

**Known Issues:** None


#### Get Schedule Details:

**Description:** Search for a Schedule by Name (Includes `"*"` wildcard for partial matches). Returns the start/end/recurrence rule and state of the schedule.

**Use Case:** Need Use Case

**Known Issues:** None


#### Get Service Level Details:

**Description:** For a given queue and interval, get the aggregate voice service level.

**Use Case:** Can be used by flow to take action based on the current service level of a queue.

**Known Issues:** Returns the raw numerator and denominator and target for the queue. Currently designed only for voice media types, but could be adapted to other media types. Uses the analytics API for permissions, verify permissions are properly set. Requires interval range input. Will need to perform the proper division in flow to divide numerator by denominator and handle any division errors.


#### Get User Count by Presence:

**Description:** For a given queue get the count of users assigned. Optionally specify a presence (Available, Busy, etc...) to filter by.

**Use Case:** Can be used by flow to take action based on the current users in a queue.Could be used in flow controls to see if multiple users are in a meeting as compared to total number of available users and make routing decisions based on that.

**Known Issues:** Typically ACD routing status is used instead of presence as calls in a queue require users to be On-Queue to get a call.


#### Get Conversation Attribute by ConversationId:

**Description:** For a conversation get the attribute value.

**Use Case:** Needed

**Known Issues:** None


#### Update Emergency Group:

**Description:** Updates an Emergency Group by ID

**Use Case:** Could be used to toggle state of emergency group via architect flow.

**Known Issues:** None


#### Update Data Table Row:

**Description:** Updates a given row of a given datatable.

**Use Case:** Example code on how to achieve data table row changes from Architect.

**Known Issues:** Need to specify the data table columns in place of "Field name 1".


#### Get On Queue Agents by Skill:

**Description:** Returns the number of agents who are members of a given queue that are assigned a given skill and know a given language that are in a given routing status.

**Use Case:** Alternative to 

**Known Issues:** Doesn't have a "catch-all" for all scenarios, for example the NULL situation.