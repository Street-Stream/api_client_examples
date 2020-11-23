# Documentation for the Street Strean API

## Introduction

The streetstream.co.uk API allows the booking and monitoring of same day couriers through the Street System.  The latest version provides support for point to point deliveries and mutlidrop deliveries where a single collection results in from two to twenty deliveries. Multidrop also allows as booked order delivery or optimisation of the route by the system.

The API is currently available only available to customers with Invoice arrangement. Please contact us at info@streetstream.net for a demo account.   

The latest version of the system also provides greatly improved near realtime tracking which is available from the API.

The API is HTTPS only and we use JWT tokens on all authenticated requests.  An example of this request is available in the Postman collection.

For monitoring of live jobs we provide the ability to register a webhook which we will call with updates on the progress of the job.

Over the coming weeks we will be expanding our GitHub examples repository at https://github.com/Street-Stream/api_client_examples The initial version will focus on examples for the [Postman](https://www.postman.com) tool.

## Reference Data

The system defines common package types, transport types and insurance levels used in the booking process.

### Package Types

For point to point jobs the booking should specify the package type code.  Definitions of the sort of package we currently handle are available from https://prod-api.streetstreamdev.co.uk/api/ref-data/package-types . Each package type defines a unique ID which is passed into the booking process.  Ensuring that an appropriate package type is selected for a booking is important to ensure that the courier arrives with appropriate transport and can successfully complete the booking.  Below is an example of one of our current package type definitions.  This provides maximum dimensions as well as a maximum weight.  The order attributes are used for group and ordering in our web interface so would not usually be of interest to API users.     

```json
{"id":"PT1003","type":"Envelope","size":"Large","maxWeightKilograms":1.0,"maxWidthCentimetres":42.0,"maxHeightCentimetres":30.0,"maxDepthCentimetres":5.0,"groupOrder":1,"orderInGroup":3,"active":true,"defaultTransportType":"BICYCLE"}
```

### Courier Transport Types

In the case of multidrop jobs the booking is made on the basis of booking a courier with an appropriate transport type.  Current transport types are available from https://prod-api.streetstreamdev.co.uk/api/ref-data/courier-transport-types . Each transport type defines the maximum dimensions and weight for all the items needing to be delivered.

```json
{"code":"UKCTT006","defaultLabel":"MEDIUM_VAN_MESSENGER","order":6,"motorised":true,"maxD1cm":70,"maxD2cm":170,"maxD3cm":120,"maxWeightKg":500}
```   
