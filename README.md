<!--
       Copyright 2017 IBM Corp All Rights Reserved

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
-->

The *stock-quote* microservice gets the price of a specified stock.  It hits an API in **API Connect**,
which drives a call to `https://cloud.iexapis.com/stable/stock/{symbol}/quote` to get the actual data,
then mediates the structure of the returned JSON as described below.  Note that an API key needs to be
passed as a query param named `token` (the API Connect impl does that for you).

It responds to a `GET /{symbol}` REST request, where you pass in a stock ticker symbol, and it returns
a JSON object containing that *symbol*, the *price*, the *date* and the *time* it was quoted.  The *time*
field is the number of milliseconds since the start of 1970 (used in determining quote staleness).

For example, if you hit the `http://<hostname>:9080/stock-quote/IBM` URL, it would return
`{"symbol": "IBM", "price": 155.23, "date": "2016-06-27", "time": 1467028800000}`.

This service uses **Redis** for caching.  When a quote is requested, it first checks to see if it is
in the cache, and if so, whether it is less that an hour old, and if so, just uses that.  Otherwise
(or if any exceptions occur communicating with Redis), it drives the REST call to **API Connect** as
usual, then adds it to **Redis** so it's there for next time.

The *Java for Redis*, or **Jedis**, library is used for communicating with **Redis**.

 ### Prerequisites for OCP Deployment
 This project requires three secrets: `jwt`, `redis`, and `urls`.
 
 ### Build and Deploy to OCP
To build `stock-quote` clone this repo and run:
```
cd templates

oc create -f stock-quote-liberty-projects.yaml

oc create -f stock-quote-liberty-deploy.yaml -n stock-quote-liberty-dev
oc create -f stock-quote-liberty-deploy.yaml -n stock-quote-liberty-stage
oc create -f stock-quote-liberty-deploy.yaml -n stock-quote-liberty-prod

oc new-app stock-quote-liberty-deploy -n stock-quote-liberty-dev
oc new-app stock-quote-liberty-deploy -n stock-quote-liberty-stage
oc new-app stock-quote-liberty-deploy -n stock-quote-liberty-prod

oc create -f stock-quote-liberty-build.yaml -n stock-quote-liberty-build

oc new-app stock-quote-liberty-build -n stock-quote-liberty-build

```
