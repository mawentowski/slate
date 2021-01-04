<!-- Fill out later after conceptual doc -->

## Request Bearer Token
## Request Session Token
## Subscribe
You are subscribing your application
Subscribe subscribes to the location contained in the session token and returns
a capability list for that location. This is a list of devices at the location and their capabilties
and also their statues
The device_ID is the device

## Status
You enter device_IDs (devices) into a device_ID list to get statuses for one or multiple devices.
device_ID I believe is the IP address on a local network that gives them internet access
You may need to list responses
Example response:
{"BOH-T1-GATE1-WKS-000.BGR1.BGR1":”OK”}
Your example is missing the actual device statues in the return response -- look at reference doc.

## Reserve - MISSING
This method reserves the peripheral for exclusive or shared access.  
Release This method releases control of a peripheral.

## Enable
Enable a peripheral for reading or scanning. Enabling a peripheral indicates that the application is ready to receive data via the messages.

## Send AEA Command 
Sends it a command defined somewhere else.

## Disable
Executing a Disable request will stop the flow of data to the application.

## Unsubscribe


You basically have scanners and printers:




Removed fields:
hasEnabled -- relates to one than one app
additionalSettings
service bus stuff
device_details
webhooks
tenant

Do response parameters need to say "mandatory"?

Improvement areas:
Many templates I have seen do not have Example Value / Model










## Request Session Token

<%= partial "includes/codesnippets/getsessiontoken.md.erb" %>

Request a session token that can be used at your specified location in order to make other API requests, most notably the Subscribe request that requires a location_ID.
The location_ID is encoded in the session token.'

Subscribe: 
<!-- Subscribe to the location (subscribes a peripheral to a location) which reserves it to the session token?
Then unsubscribe to release all peripherals reserved by session token? -->

Subscribe to the location encoded in your session token. The only required action is to include it in the Request body, 
as it is decoded and checked during the Subscribe call.

<!-- This returns the last known status of the logical type Scanner with the capability to read barcodes. -->



## Status

Request the status of peripherals.  

This is an information command and the status will still be returned even if the peripheral is acquired for exclusive access by another application. If the status of all peripherals is desired, leave the device_list empty.

### HTTP Request

`POST location/status`

### Request Body
For the sake of exercising the API, you can specify any numerical value as the transaction ID and view it via the subscription.

Param | Type | O/M | Description
------|------|-----|------------
device_list | string | Mandatory | Retrieves the real-time status of the peripherals identified in the device_list.
transaction_ID | string | Optional | The transaction_ID used by the application.
session_token | string | Optional | The session_token received from the previously executed Subscribe request.

<aside class="success">
A successful request returns a status of 200 OK and the status message will either be sent to 1) the endpoint defined by the return payload from the Subscribe request for the specified location, or 2) to the service bus. The second scenario occurs when a webhook was not specified in the previously executed Subscribe request.
</aside>


## Enable

<%= partial "includes/codesnippets/enable.md.erb" %>

Enable a peripheral for reading or scanning.

Enabling a peripheral indicates that the application is ready to receive data via the messages.

Executing a Disable request will stop the flow of data to the application.

There is no need to explicitly acquire the device before this command is issued, but in this case the ExclusiveAccess flag will be set to True by default.

The application should disable the device when it receives an AccessExpiry message with a ‘AccessRequested’ parameter.  And should disable the device when not in use.

### HTTP Request

`POST peripheral/enable`

### Request Body

Param | Type | O/M | Description
------|------|-----|------------
device_ID | string | Mandatory | The location of the peripheral.
transaction_ID | string | Optional | The transaction_ID used by the application.
session_token | string | Optional | The session_token received from the previously executed Subscribe request.


<aside class="success">
A successful request returns a 200 OK Status and the peripheral is now enabled for scanning and reading.
</aside>

### Response

No JSON response.

<aside class="warning">
An unsuccessful request may return either a 400, 403, or 501 error code with an explanatory message. For the general meaning of these responses, see the Error Codes section of this document.
</aside>


## Send AEA Command 

<%= partial "includes/codesnippets/sendaeacommand.md.erb" %>

Send an AEA command to a peripheral.

### HTTP Request

`POST peripheral/aea/command`

### Request Body

Param | Type | O/M | Description
------|------|-----|------------
device_ID | string | Mandatory | The location of the peripheral.
transaction_ID | string | Optional | The transaction_ID used by the application.
request_ID | string | Mandatory | A unique ID for the command to ensure the same command is not executed multiple times.
session_token | string | Optional | The session_token received from the previously executed Subscribe request.
command | string | Mandatory | An AEA command.

<aside class="success">
A successful request returns a 200 OK Status indicating the AEA message was successfully sent. 
</aside>

### Response

No JSON response.

<aside class="warning">
An unsuccessful request may return either a 400 or 401 error code with an explanatory message. For the general meaning of these responses, see the Error Codes section of this document.
</aside>

## Disable

<%= partial "includes/codesnippets/disable.md.erb" %>

Send a Disable request whenever a device is no longer required.
This request cancels a previously executed Enable request and 
prevents the return of data from the device_ID specified.  
It does not release the peripheral if the Release bool is set to False.  
If the peripheral is not disabled, this request will disable the peripheral. 

### HTTP Request

`POST peripheral/disable`

### Request Body

Param | Type | O/M | Description
------|------|-----|------------
device_ID | string | Mandatory | The location of the peripheral.
transaction_ID | string | Mandatory | Sent by the application.
session_token | string | Mandatory | The session token received from the subscription call.

<aside class="success">
A successful request returns a 200 OK Status if the Disable request succeeds, or if there is no outstanding Enable request.
</aside>

### Response

No JSON response.

<aside class="warning">
An unsuccessful request may return a 400 error code with an explanatory message. For the general meaning of these responses, see the Error Codes section of this document.
</aside>

## Unsubscribe

<%= partial "includes/codesnippets/unsubscribe.md.erb" %>

Release all peripherals reserved by the session and disable (or cancel) all outstanding commands. This applies only to the application associated with the session token.

### HTTP Request

`POST location/unsubscribe`

### Request Body

Param | Type | O/M | Description
------|------|-----|------------
session_token | string | Mandatory | The location at which the session token will be used.

<aside class="success">
A successful request returns a status of 204: No content.
</aside>

### Response
No JSON returned.

<aside class="warning">
An unsuccessful request may return either a 400, 401, or a 409 error code with an explanatory message. For the general meaning of these responses, see the Error Codes section of this document.
</aside>



