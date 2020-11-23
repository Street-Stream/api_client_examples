# Documentation for the Street Strean API

## Introduction

The streetstream.co.uk API allows the booking and monitoring of same day couriers through the Street System.  The latest version provides support for point to point deliveries and mutlidrop deliveries where a single collection results in from two to twenty deliveries. Multidrop also allows as booked order delivery or optimisation of the route by the system.

The API is currently available only available to customers with Invoice arrangement. Please contact us at info@streetstream.net for a demo account.   

The latest version of the system also provides greatly improved near realtime tracking which is available from the API.

The API is HTTPS only and we use JWT tokens on all authenticated requests.

For monitoring of live jobs we provide the ability to register a webhook which we will call with updates on the progress of the job.

Over the coming weeks we will be expanding our GitHub examples repository at https://github.com/Street-Stream/api_client_examples The initial version will focus on examples for the [Postman](https://www.postman.com) tool.
