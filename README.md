# Documentation for the Street Stream API

## Introduction

The streetstream.co.uk API allows the booking and monitoring of same day courier deliveries through the Street Stream System.  The latest version provides support for point to point deliveries and mutlidrop deliveries where a single collection results in from two to twenty deliveries. Multidrop also allows as booked order delivery or optimisation of the route by the system.

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
{"id":"PT1003","type":"Envelope","size":"Large",
  "maxWeightKilograms":1.0,
  "maxWidthCentimetres":42.0,"maxHeightCentimetres":30.0,"maxDepthCentimetres":5.0,
  "groupOrder":1,"orderInGroup":3,
  "active":true,"defaultTransportType":"BICYCLE"
}
```

### Courier Transport Types

In the case of multidrop jobs the booking is made on the basis of booking a courier with an appropriate transport type.  Current transport types are available from https://prod-api.streetstreamdev.co.uk/api/ref-data/courier-transport-types . Each transport type defines the maximum dimensions and weight for all the items needing to be delivered.

```json
{"code":"UKCTT006","defaultLabel":"MEDIUM_VAN_MESSENGER",
  "order":6,"motorised":true,
  "maxD1cm":70,"maxD2cm":170,"maxD3cm":120,
  "maxWeightKg":500
}
```   

### Insurance Cover Levels

Street Stream allows customers to specify different levels of insurance cover on a per booking basis.  Current insurance levels can be retrieved from https://prod-api.streetstreamdev.co.uk/api/ref-data/insurance-levels . Below shows the data for our professional level which comes with a VAT exclusive additional cost of £1 and provides cover up to £1,000 per booking.  If cover level is not specified it defaults to our personal cover level with no additional cost and cover of up to £100 per booking.     

```json
{"defaultLabel":"PROFESSIONAL",
  "code":"UKIC0002",
  "exVatCost":1.00,"payableVat":0.20,"totalPayableWithVat":1.20,
  "maxCover":1000,"currencyCode":"GBP"
}
```

### Offer Acceptance Strategy

Selecting an offer acceptance strategy allows our customers to determine which of the couriers who offer to take on the job we match their booking to. Currently this is a fairly short list.

#### AUTO_CLOSEST_COURIER_TO_ME

We track where the courier is when they make an offer to carryout a job and determine their distance to the first pickup.  The job is then given to the courier can get to you in the shortest amount of time.  Typically this is used for very urgent jobs.

#### AUTO_FAVOURITE_COURIER

Our customers often build long term relationships with couriers undertaking their deliveries.  Marking a courier they have previously used as a favourite means we automatically select that courier if they are available. 

#### AUTO_HIGHEST_RATED_COURIER

Customers have the opportunity to rate the courier for each job booked through Street Stream.  By using this option we simply select the courier with the highest average rating for the job.
