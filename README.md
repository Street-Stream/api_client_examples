# Documentation for the Street Stream API

## Introduction

The streetstream.co.uk API allows the booking and monitoring of same day courier deliveries through the Street Stream System.  The latest version provides support for point to point deliveries and mutlidrop deliveries where a single collection results in from two to twenty deliveries. Multidrop also allows as booked order delivery or optimisation of the route by the system.

The API is currently available only available to customers with Invoice arrangement. Please contact us at info@streetstream.net for a demo account.   

The latest version of the system also provides greatly improved near realtime tracking which is available from the API.

The API is HTTPS only and we use JWT tokens on all authenticated requests.  An example of this request is available in the Postman collection.

For monitoring of live jobs we provide the ability to register a webhook which we will call with updates on the progress of the job.

Over the coming weeks we will be expanding our GitHub examples repository at https://github.com/Street-Stream/api_client_examples The initial version will focus on examples for the [Postman](https://www.postman.com) tool.

## Authenticating

In order to make calls to the Street Stream API A bearer token must be included with each request as an Authorization header. A Bearer token can be created by posting a payload as below to ***/api/tokens***.  The bearer token will then be returned in the response Authorization header.  Bearer tokens are valid for twenty four hours.

```json
{
"email": "john@example.org", "authType": "CUSTOMER","password": "MY_PASSWORD"
}

```

Some URLs also require your unique customer ID. If you don't know your customer ID it can be retreived by making a GET request to **/api/users/customer/whoami**  Which will return somethings like the below.

```json
{
 "customerId":"83cd7db6-5051-493a-8c52-f1be0ee8add7", 
 "registeredEmail":"john@example.orgm"
}
```

## Street Stream Environments

In addition to our production environment we provide a staging environment which can be used for testing.  You can self regsiter for an account but as mentioned we will need to enable invoicing.

###Staging

* https://stage-api.streetstreamdev.co.uk      **API**
* https://staging.streetstream.co.uk **Customer web app**

### Production 
  
* https://prod-api.streetstreamdev.co.uk      **API**
* https://streetstream.co.uk **Customer web app**


## Reference Data

The system defines common package types, transport types and insurance levels used in the booking process.  Requests to retrieve reference data do not require a bearer token.

### Package Types

For point to point jobs the booking should specify the package type code.  Definitions of the sort of package we currently handle are available from https://prod-api.streetstreamdev.co.uk/api/ref-data/package-types . Each package type defines a unique ID which is passed into the booking process.  Ensuring that an appropriate package type is selected for a booking is important to ensure that the courier arrives with appropriate transport and can successfully complete the booking.  Below is an example of one of our current package type definitions.  This provides maximum dimensions as well as a maximum weight.  The order attributes are used for group and ordering in our web interface so would not usually be of interest to API users.     

```json
{
  "id":"PT1003","type":"Envelope","size":"Large",
  "maxWeightKilograms":1.0,
  "maxWidthCentimetres":42.0,"maxHeightCentimetres":30.0,"maxDepthCentimetres":5.0,
  "groupOrder":1,"orderInGroup":3,
  "active":true,"defaultTransportType":"BICYCLE"
}
```

### Courier Transport Types

In the case of multidrop jobs the booking is made on the basis of booking a courier with an appropriate transport type.  Current transport types are available from https://prod-api.streetstreamdev.co.uk/api/ref-data/courier-transport-types . Each transport type defines the maximum dimensions and weight for all the items needing to be delivered.

```json
{ 
  "code":"UKCTT006","defaultLabel":"MEDIUM_VAN_MESSENGER",
  "order":6,"motorised":true,
  "maxD1cm":70,"maxD2cm":170,"maxD3cm":120,
  "maxWeightKg":500
}
```   

### Insurance Cover Levels

Street Stream allows customers to specify different levels of insurance cover on a per booking basis.  Current insurance levels can be retrieved from https://prod-api.streetstreamdev.co.uk/api/ref-data/insurance-levels . Below shows the data for our professional level which comes with a VAT exclusive additional cost of £1 and provides cover up to £1,000 per booking.  If cover level is not specified it defaults to our personal cover level with no additional cost and cover of up to £100 per booking.     

```json
{ 
  "defaultLabel":"PROFESSIONAL",
  "code":"UKIC0002",
  "exVatCost":1.00,"payableVat":0.20,"totalPayableWithVat":1.20,
  "maxCover":1000,"currencyCode":"GBP"
}
```

### Offer Acceptance Strategy

Selecting an offer acceptance strategy allows our customers to determine which of the couriers who offer to take on the job we match their booking to. Currently this is a fairly short list.

#### AUTO CLOSEST COURIER TO ME

We track where the courier is when they make an offer to carry out a job and determine their distance to the first pickup.  The job is then given to the courier can get to you in the shortest amount of time.  Typically this is used for very urgent jobs.

#### AUTO FAVOURITE COURIER

Our customers often build long term relationships with couriers undertaking their deliveries.  Marking a courier they have previously used as a favourite means we automatically select that courier if they are available. 

#### AUTO HIGHEST RATED COURIER

Customers have the opportunity to rate the courier for each job booked through Street Stream.  By using this option we simply select the courier with the highest average rating for the job.

## Job Status, Pick Up Status and Drop Off Status

Every job, pick up and drop off has a single status and a history of statuses that have been applied. This section details all possible statuses and wether the customer will be notified of a particualr status being applied if they have registered a webhook with Street Strem, see the Webhook section for details.  

Job Status    | Notified by Webhook |  Description
------------- | --------------------|------------------
DRAFT			 |  No                 | Intial status of job when created.  Where submit immediately is true will transition directly to SUBMITTED FOR OFFERS.  Draft jobs are not yet visible to couriers and no quotes will be received.
SUBMITTED FOR OFFERS  | No				| The job is visible to couriers who can quote the job.
OFFERS RECEIVED | Yes | Triggered by at least one courier offering to undertake the job.
OFFER SELECTED | No | One of the courier quotes has been selected.  In the case of API jobs this is triggered by the acceptance algorithm.
JOB AGREED | Yes | The details of the job including payment and which courier will undertake the job have been agreed. Since obth sides are now committed cancellation after this point will incurr a cancellation charge.
CANCELLED BEFORE ARRIVAL | No | Customer has requested cancellation prior to the courier arriving. A cancellation fee will apply. This is a terminal state for the job so no further status updates should be expected.
IN PROGRESS | Yes | Triggered when a courier arrives at the first collection.
COMPLETED SUCESSFULLY | Yes | All packages have been delivered successfully. Terminal state.
FAILED | Yes | All | The job has failed.  This will be triggered if the collection fails.  A seperate pick up status notification for the pick up will also be received to indicate which pick up and why it failed.
ADMIN_CANCELLED | Yes | A Street Stream administrator needed to cancel the job.  

Additonally there is **AUTHORISE CARD and EXPIRED DURING PAYMENT**. This could be seen on historical jobs booked through our web interface and paid for by credit card.  These would not occur on API jobs with invoicing.

#### Pick Up Status

Pick Up Status    | Notified by Webhook |  Description
------------- | --------------------|------------------
NOT YET COLLECTED | No | Initial status when job is created.
CUSTOMER CANCELLED BEFORE ARRIVAL | No | Customer has cancelled the job the pick up was part of.
ARRIVED AT COLLECTION | Yes | The courier hjas arrived at the location of the pick up.
COLLECTED | Yes | Collection has been made successfully. Terminal state.
NOT AS DESCRIBED | Yes | Courier could not accept the package as it did not comply with the packae type in the booking.  For example was too heavy for a cycle courier to accept. Terminal state.
NO RESPONSE | Yes | The courier was unable to get answer using the provided address and contact details.  Couriers can only apply this status once at least 15 minutes has passed since they applied ARRIVED AT COLLECTION. Terminal state
CUSTOMER CANCELLED ON ARRIVAL | No | Customer triggered cancellation after courier arrives.

### Drop Off Status


Drop Off Status    | Notified by Webhook |  Description
------------- | --------------------|------------------
NOT YET COLLECTED | No | Initial status when job is created.
COLLECTED | No | Pick up for this drop has been carred out successfully.
CANCELLED | No | Drop has been cancelled possibly due to a failure to pick up or cancellation.
ARRIVED AT DELIVERY | Yes | Courier has arrived at the provided address.
DELIVERY ATTEMPT FAILED | Yes | Courier was unable to deliver the package on this attempt.  Can occur mutiple times.
DELIVERED | Yes | Package was succssfulyl deliverd.  Notes and singature if requested will now be avaialble.  


## Creating a Point to Point Job for a Package

Below is an example taken fron the Postman collection requesting a point to point job for package code PT1003 (an envelope).  The job is to be submitted immediately with the Street Stream system selecting the highest rated courier able to do the job. All timestamps must use UTC.  Most text fields are limited to a lenght of 255 with the exception of notes which allow up to 500.  Address two , county and country can be provided ommitted or null.  Address one can be used to contain business and first line for example 'Blogs and Blogs Ltd, 12 Mornington Crescent' as one line.

```json
{
    "offerAcceptanceStrategy": "AUTO_HIGHEST_RATED_COURIER",
    "packageTypeId": "PT1003",
    "jobLabel": "My first test job",
    "insuranceCover": "CORPORATE",
    "submitForQuotesImmediately": true,
    "pickUp": {
        "contactNumber" : "020 7754 5452",
        "contactName" : "Jane Doe",
        "addressOne" : "A Company HQ",
        "addressTwo" : "11 Claylands Road",
        "city" : "London",
        "county" : "",
        "postcode" : "SW8 1NL",
        "pickUpNotes": "notes about the pickup",
        "pickUpFrom": "2020-11-26T14:34:37.339285Z",
        "pickUpTo": "2020-11-26T15:04:37.339319Z"
    },
    "dropOff": {
        "contactNumber" : "",
        "contactName" : "S Holmes",
        "addressOne" : "221b Baker Street",
        "city" : "London",
        "postcode" : "NE1 6XE",
        "dropOffFrom": "2020-12-26T16:04:37.339589Z",
        "dropOffTo": "2020-11-26T17:04:37.339593Z",
        "clientTag": "ORDER-123",
        "deliveryNotes": "Use the side alley ring second bell"
    }
}
```

### Reponse

After sending a ***POST*** which results in the successful creation of a job the Street Stream system will redirect to the location for the new job from which can be used to get the details of the create job. Clients can either follow the redirect or retreive the Location header from the response. The returned location will look something like /api/job/pointtopoint/eb81d193-97e0-498f-a4f4-b248f5c0f7 for a point to point job.

Issue a GET request will return something similar to the below. The returned data contains the data provided at booking time with the addition of the following. 

* Job pricing broken down to show courier cost, insturance cost and applicable VAT
* An estimate of the distance required to get from the pick up to the drop off and an estimate of how long that will take
* A unique identifier for the job and for each pick up or drop off
* Coordinates for each pick up or drop off 
* The history of statuses applied to the job
* The courier id which will be null until the job is agreed
* Quotes received for the job if any
* The quote that has been selected if any

```json
{
    "id": "6d4f4a91-18c2-491e-89df-c97edf9775df",
    "offerAcceptanceStrategy": "AUTO_HIGHEST_RATED_COURIER",
    "packageTypeId": "PT1003",
    "courierTransportTypeRequested": "CARGO_BIKE",
    "jobLabel": "My first test job",
    "jobExpiry": "2020-11-26T15:04:37.339319Z",
       "jobCharge": {
        "exVatTotalPriceBeforeInsurance": 13.27,
        "exVatInsurance": 2.00,
        "payableVat": 3.05,
        "totalPayableWithVat": 18.32,
        "currencyCode": "GBP",
        "discountAmount": 0.00
    },
    "jobStatus": "SUBMITTED_FOR_OFFERS",
    "jobCreated": "2020-11-26T11:18:00.894459Z",
    "jobLastModified": "2020-11-26T11:18:01.089617Z",
    "courierId": null,
    "feedbackProvided": false,
    "confirmedPaymentMechanism": "INVOICE",
    "insuranceCover": "CORPORATE",
    "estimatedRouteDistanceKm": 8.36,
    "estimatedRouteTimeSeconds": 2733,
    "quotes": [],
    "selectedQuote": null,
    "pickUp": {
        "id": "2d8c9a0e-8bed-4568-9629-48cd766e5e25",
        "addressOne": "A Company HQ",
        "addressTwo": "11 Claylands Road",
        "city": "London",
        "county": "",
        "country": null,
        "postcode": "SW8 1NL",
        "contactNumber": "020 7754 5452",
        "contactName": "Jane Doe",
        "pickUpNotes": "notes about the pickup",
        "pickUpFrom": "2020-11-26T14:34:37.339285Z",
        "pickUpTo": "2020-11-26T15:04:37.339319Z",
        "routeStopNumber": 0,
        "clientTag": null,
        "save": false,
        "addressBookId": null,
        "latitude": 51.4807417,
        "longitude": -0.1147992
    },
    "dropOff": {
        "id": "97d21180-a0ba-40d2-bb3c-7a0d44c05590",
        "addressOne": "221b Baker Street",
        "addressTwo": null,
        "city": "London",
        "county": null,
        "country": null,
        "postcode": "NE1 6XE",
        "contactNumber": "",
        "contactName": "S Holmes",
        "dropOffNotes": null,
        "dropOffFrom": "2020-12-26T16:04:37.339589Z",
        "dropOffTo": "2020-11-26T17:04:37.339593Z",
        "signatureRequired": false,
        "signatureFileLocation": null,
        "routeStopNumber": 1,
        "clientTag": "ORDER-123",
        "deliveryNotes": "Use the side alley ring second bell",
        "save": false,
        "addressBookId": null,
        "latitude": 51.523767,
        "longitude": -0.1585557
    },
    "jobType": "POINT_TO_POINT",
    "statusHistory": [
        {
            "jobStatus": "DRAFT",
            "appliedAt": "2020-11-26T11:18:00.894498Z",
            "appliedBy": "bob@example.org:[CUSTOMER]"
        },
        {
            "jobStatus": "SUBMITTED_FOR_OFFERS",
            "appliedAt": "2020-11-26T11:18:01.079495Z",
            "appliedBy": "bob@example.org:[CUSTOMER]"
        }
    ],
    "insuranceSelectedAt": "2020-11-26T11:18:00.894440Z",
    "isTerminal": false
}
```

## Creating a Point to Point Job Specifying a Transport Type

In the case where a point to point job specifies a package the Street Stream system will then determine an appropriate transport type based on dimensions and weight.  This in turn determines which couriers we notify and which couriers we will allow to take on the job.  There may be occassions where there are other considerations beyond dimensions and weight, for example fragility.  Where customers wish to have more control for example such as the case of delviering a cake where a car or van may be a better choice it is possible to skip the package parameter and specify the transport type directly as in the example below.


```json
{
    "offerAcceptanceStrategy": "AUTO_HIGHEST_RATED_COURIER",
    "courierTransportType": "PALLET_CARRIER_MESSENGER",
    "jobLabel": "My first test job",
    "insuranceCover": "CORPORATE",
    "submitForQuotesImmediately": true,
    "pickUp": {
        "contactNumber" : "020 7754 5452",
        "contactName" : "Jane Doe",
        "addressOne" : "A Company HQ",
        "addressTwo" : "11 Claylands Road",
        "city" : "London",
        "county" : "",
        "postcode" : "SW8 1NL",
        "pickUpNotes": "notes about the pickup",
        "pickUpFrom": "2021-05-12T14:34:37.339285Z",
        "pickUpTo": "2021-05-12T15:04:37.339319Z"
    },
    "dropOff": {
        "contactNumber" : "",
        "contactName" : "S Holmes",
        "addressOne" : "221b Baker Street",
        "city" : "London",
        "postcode" : "NE1 6XE",
        "dropOffFrom": "2021-05-12T16:04:37.339589Z",
        "dropOffTo": "2021-05-12T17:04:37.339593Z",
        "clientTag": "ORDER-123",
        "deliveryNotes": "Use the side alley ring second bell"
    }
}
``` 

The response below shows the one difference between specifying transport type and package.  Where transport type is specified package will be null.  Where package type is specified we will record the default transport type determined by the system.

```json
{
    "id": "c687119c-03b0-4992-922f-34ec3f0f654d",
    "offerAcceptanceStrategy": "AUTO_HIGHEST_RATED_COURIER",
    "packageTypeId": null,
    "courierTransportTypeRequested": "PALLET_CARRIER_MESSENGER",
    "jobLabel": "My first test job",
    "jobExpiry": "2020-05-12T15:04:37.339319Z",
    "jobCharge": {
        "exVatTotalPriceBeforeInsurance": 28.78,
        "exVatInsurance": 2.00,
        "payableVat": 6.16,
        "totalPayableWithVat": 36.94,
        "currencyCode": "GBP",
        "discountAmount": 0.00
    },
    "jobStatus": "SUBMITTED_FOR_OFFERS",
    "jobCreated": "2021-05-11T04:57:14.079803Z",
    "jobLastModified": "2021-05-11T04:57:14.476246Z",
    "courierId": null,
    "feedbackProvided": false,
    "confirmedPaymentMechanism": "INVOICE",
    "insuranceCover": "CORPORATE",
    "estimatedRouteDistanceKm": 7.28,
    "estimatedRouteTimeSeconds": 2026,
    "quotes": [],
    "selectedQuote": null,
    "pickUp": {
        "id": "479200ae-0cec-468f-a2d4-f17d16a512c6",
        "addressOne": "A Company HQ",
        "addressTwo": "11 Claylands Road",
        "city": "London",
        "county": "",
        "country": null,
        "postcode": "SW8 1NL",
        "contactNumber": "020 7754 5452",
        "contactName": "Jane Doe",
        "pickUpNotes": "notes about the pickup",
        "pickUpFrom": "2021-05-12T14:34:37.339285Z",
        "pickUpTo": "2020-05-12T15:04:37.339319Z",
        "routeStopNumber": 0,
        "clientTag": null,
        "save": false,
        "addressBookId": null,
        "latitude": 51.4807417,
        "longitude": -0.1147992
    },
    "dropOff": {
        "id": "17c935ae-3f2b-4bf1-a35e-dba844ed7112",
        "addressOne": "221b Baker Street",
        "addressTwo": null,
        "city": "London",
        "county": null,
        "country": null,
        "postcode": "NE1 6XE",
        "contactNumber": "",
        "contactName": "S Holmes",
        "dropOffNotes": null,
        "dropOffFrom": "2020-05-12T16:04:37.339589Z",
        "dropOffTo": "2020-05-12T17:04:37.339593Z",
        "signatureRequired": false,
        "signatureFileLocation": null,
        "routeStopNumber": 1,
        "clientTag": "ORDER-123",
        "deliveryNotes": "Use the side alley ring second bell",
        "save": false,
        "addressBookId": null,
        "latitude": 51.523767,
        "longitude": -0.1585557
    },
    "jobType": "POINT_TO_POINT",
    "statusHistory": [
        {
            "jobStatus": "DRAFT",
            "appliedAt": "2021-05-11T04:57:14.080169Z",
            "appliedBy": "jonas.partner@gmail.com:[CUSTOMER]"
        },
        {
            "jobStatus": "SUBMITTED_FOR_OFFERS",
            "appliedAt": "2021-05-11T04:57:14.283960Z",
            "appliedBy": "jonas.partner@gmail.com:[CUSTOMER]"
        }
    ],
    "insuranceSelectedAt": "2021-05-11T04:57:14.079768Z",
    "isTerminal": false
}
```




## Getting Notified About Status Updates

Customers can register a webhook which will be called each time an event that is notifiable is applied to a job, drop or pick up. 

### Registering the webhook

Registering a webhook can be done by sending a post request to **/api/users/customer/webhook/{{CUSTOMER_ID}}**.

```json
{
    "url":"https://mywebhook.example.org"
}
``` 

This will return if the webhook is HTTPS and could be invoked successfully.  To test the webhook we make a POST request with below payload.  The reponse to the test POST needs to be 200 in order for the webhook to be successfully regsitered.

```json
{
    "type":"test","message":"Hello From Street Stream"
}
```

### Status Updates

Examples of status updates sent to registered webhooks are shown below.  The main difference is wheter they include a stopId.

A job that has been fully completed would result in.

```json
{
   "notificationType":"JOB",
   "jobId":"84bc9c2ebbb6-49dc-8e1fbc62e66a71e2",
   "status":"COMPLETED_SUCCESSFULLY","stopId":null,
   "appliedAt":"2020-11-26T09:59:29.948136Z"
}
```

Courier arrived for pick up

```json
{
   "notificationType":"JOB",
   "jobId":"84bc9c2e-bbb6-49dc-8e1f-bc62e66a71e2",
   "status":"ARRIVED_AT_COLLECTION",
   "stopId":"ed785e83-d9c0-4a30-b8ec-677d08e1178a",
   "appliedAt":"2020-11-26T09:58:37.396791Z"
}
```

Succesful collection

```json
{
    "notificationType":"JOB",
    "jobId":"84bc9c2e-bbb6-49dc-8e1f-bc62e66a71e2",
    "status":"COLLECTED",
    "stopId":"ed785e83-d9c0-4a30-b8ec-677d08e1178a",
    "appliedAt":"2020-11-26T09:58:39.179154Z"
}
```

Successful delivery

```json
{
   "notificationType":"JOB",
   "jobId":"84bc9c2e-bbb6-49dc-8e1f-bc62e66a71e2",
   "status":"DELIVERED",
   "stopId":"a6e3efd8-4c3f-417c-8e91-a2adce629e54",
   "appliedAt":"2020-11-26T09:59:29.483869Z"
}
```


## Tracking Courier Location

A location report for a job can be retreived by calling GET on ***/api/location/{{JOB_ID}}*** . The loction report includes the current location of the courier undertaking the job along with the ID for the next planned stop and an estaimated time of arrival.  An example reponse is shown below.  The location report is avaialable from thirty minutes before the first pick up window starts and remains available until the job completes.  Requesting a location report for a job that does not meet those requirements will result in a 404.


```json
{
    "courierLatestLatitude": 51.4052677,
    "courierLatestLongitude": -0.1003478,
    "locationTakenUTC": "2020-11-27T07:37:52Z",
    "nextStop": {
        "courierToNextStopDistanceKm": 1.7,
        "stopId": "327de305-0765-4558-923d-b8d3aff7046d",
        "stopEta": "2020-11-30T23:04:56.246380Z",
        "stopLatitude": 51.50527779999999,
        "stopLongitude": -0.1002778
    }
}

``` 

## Create a Multi Drop Job

Below is a simple example of the creation of a job with one pick up and two drops.  The obivous difference is we have an array of drops rather than a single dropOff.  Also multi drop drops don't have an individual from and to time.  Instead the job has a top level deliveryFrom and deliveryTo tine window.  This represents the earliest time that a courier should start delivering and time by which they should finish the curcuit.  Additionally there is a top level optimise route attribute which is set to true.  Where this is a true the system will optimise the route plan for the orders rather that being based on the order provided.  Where a return to base job is required the request can include the pick up address as the last drop.  Route optimisation will detect this and only optimise the order of the other drops.

```json
{
    "offerAcceptanceStrategy": "AUTO_HIGHEST_RATED_COURIER",
    "jobLabel": "My first test multidrop job",
    "insuranceCover": "CORPORATE",
    "submitForQuotesImmediately": true,
    "optimiseRoute":true,
    "courierTransportType": "MEDIUM_VAN_MESSENGER",
    "deliveryFrom":"2020-12-09T09:00:00.000000Z",
    "deliveryTo": "2020-12-09T10:00:00.000000Z",
    "pickUp": {
        "contactNumber": "020 7754 5452",
        "contactName": "Jane Doe",
        "addressOne": "A Company HQ",
        "addressTwo": "11 Claylands Road",
        "city": "London",
        "county": "",
        "postcode": "SW8 1NL",
        "pickUpNotes": "please call when you arrive at gate",
        "pickUpFrom": "2020-12-09T08:00:00.000000Z",
        "pickUpTo": "2020-12-09T09:00:00.000000Z"
    },
    "drops": [
        {
            "contactNumber": "",
            "contactName": "J Middleton",
            "addressOne": "Runway East, 10 Finsbury Square",
            "city": "London",
            "postcode": "EC2A 1AF",
            "clientTag": "ORDER-123",
             "deliveryNotes": "leave with concierge if needed"
        },
        {
            "contactNumber": "+44 7700 900796",
            "contactName": "S Holmes",
            "addressOne": "221b Baker Street",
            "city": "London",
            "postcode": "NE1 6XE",
            "clientTag": "ORDER-456",
            "deliveryNotes": "Use the side alley ring second bell"
        }
    ]
}
```

### Successful creation response

Below is the response for getting the created job using the redirect URL returned in response to the POST request.  You will see in addition to the submitted data Street Stream has geocoded locations, added pricing and added estiamte of time and distance for the job.

Each drop now has a ***routeStopNumber*** to indicate the planned order.  In this case the two drops have been reordered from the order with which they were sent.


```json
{
    "id": "d92148e7-381e-4cb7-95be-8424e27fcb61",
    "offerAcceptanceStrategy": "AUTO_HIGHEST_RATED_COURIER",
    "courierTransportType": "MEDIUM_VAN_MESSENGER",
    "jobLabel": "My first multidrop test job",
    "jobExpiry": "2020-12-09T09:00:00Z",
    "jobType": "MULTI_DROP",
    "jobCharge": {
        "exVatTotalPriceBeforeInsurance": 38.90,
        "exVatInsurance": 2.00,
        "payableVat": 8.18,
        "totalPayableWithVat": 49.08,
        "currencyCode": "GBP",
        "discountAmount": 0.00
    },
    "jobStatus": "SUBMITTED_FOR_OFFERS",
    "jobCreated": "2020-12-08T11:06:49.473652Z",
    "jobLastModified": "2020-12-08T11:06:50.381632Z",
    "courierId": null,
    "feedbackProvided": false,
    "confirmedPaymentMechanism": "INVOICE",
    "insuranceCover": "CORPORATE",
    "estimatedRouteDistanceKm": 13.50,
    "estimatedRouteTimeSeconds": 4101,
    "quotes": [],
    "selectedQuote": null,
    "pickUp": {
        "id": "543bc907-3ffa-42b0-b7c7-b83bcb804064",
        "addressOne": "A Company HQ",
        "addressTwo": "11 Claylands Road",
        "city": "London",
        "county": "",
        "country": null,
        "postcode": "SW8 1NL",
        "contactNumber": "020 7754 5452",
        "contactName": "Jane Doe",
        "pickUpNotes": "please call when you arrive at gate",
        "pickUpFrom": "2020-12-09T08:00:00Z",
        "pickUpTo": "2020-12-09T09:00:00Z",
        "routeStopNumber": 0,
        "clientTag": null,
        "save": false,
        "addressBookId": null,
        "latitude": 51.4807417,
        "longitude": -0.1147992
    },
    "drops": [
        {
            "id": "1fe89c85-65f3-4013-aa3e-3be228d368e6",
            "addressOne": "221b Baker Street",
            "addressTwo": null,
            "city": "London",
            "county": null,
            "country": null,
            "postcode": "NE1 6XE",
            "contactNumber": "+44 7700 900796",
            "contactName": "S Holmes",
            "dropOffNotes": null,
            "dropOffFrom": null,
            "dropOffTo": null,
            "signatureRequired": false,
            "signatureFileLocation": null,
            "routeStopNumber": 1,
            "clientTag": "ORDER-456",
            "deliveryNotes": "Use the side alley ring second bell",
            "save": false,
            "addressBookId": null,
            "latitude": 51.523767,
            "longitude": -0.1585557
        },
        {
            "id": "062482df-160e-4743-82e5-a30859c3f71b",
            "addressOne": "Runway East, 10 Finsbury Square",
            "addressTwo": null,
            "city": "London",
            "county": null,
            "country": null,
            "postcode": "EC2A 1AF",
            "contactNumber": "",
            "contactName": "J Middleton",
            "dropOffNotes": null,
            "dropOffFrom": null,
            "dropOffTo": null,
            "signatureRequired": false,
            "signatureFileLocation": null,
            "routeStopNumber": 2,
            "clientTag": "ORDER-123",
            "deliveryNotes": "leave with concierge if needed",
            "save": false,
            "addressBookId": null,
            "latitude": 51.521224,
            "longitude": -0.08730059999999999
        }
    ],
    "statusHistory": [
        {
            "jobStatus": "DRAFT",
            "appliedAt": "2020-12-08T11:06:49.473728Z",
            "appliedBy": "jonas.partner@gmail.com:[CUSTOMER]"
        },
        {
            "jobStatus": "SUBMITTED_FOR_OFFERS",
            "appliedAt": "2020-12-08T11:06:50.181901Z",
            "appliedBy": "jonas.partner@gmail.com:[CUSTOMER]"
        }
    ],
    "deliveryFrom": "2020-12-09T09:00:00Z",
    "deliveryTo": "2020-12-09T10:00:00Z",
    "insuranceSelectedAt": "2020-12-08T11:06:49.473647Z",
    "isTerminal": false
}

```

 


   
  
 

