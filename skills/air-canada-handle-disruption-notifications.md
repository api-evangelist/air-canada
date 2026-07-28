---
name: Receive and handle Air Canada OrderChangeNotification (OCN) disruptions
description: >-
  Register a webhook for Air Canada-initiated Order changes, interpret the
  published status codes, and reconcile the affected Order correctly.
api: air-canada:air-canada-ndc-orderchangenotification-api
grounded_in:
  - https://ndc.aircanada.com/api/documentation/orderchangenotification
  - https://ndc.aircanada.com/api/documentation/orderretrieve
operations: [OrderChangeNotif, OrderRetrieveRQ]
messages:
  - OrderChangeNotif (inbound, Air Canada to seller)
  - OrderRetrieveRQ / OrderViewRS
---

# Receive and handle Air Canada OCN disruptions

OrderChangeNotification is the only push surface Air Canada publishes. It is
opt-in: "Such messages will only be sent to sellers that have registered with AC
to receive OCN messages."

## Register (one time, out of band)

There is no self-service webhook UI. Send an encrypted email — or share a secure
location — with `ACDirectconnectNDC@aircanada.ca` containing:

- Webhook host and webhook URL
- Authentication method (OAuth)
- AT system credentials, and PRD system credentials (optional for testing,
  required for go live)
- OAuthTokenURL and OAuthScope, if OAuth is the authentication method
- Any headers required in the request

Air Canada then runs connectivity testing before enabling delivery. Note that
Air Canada publishes **no signature scheme** for OCN — authenticity rests on the
credentials you hand over, so terminate the webhook on an endpoint that
authenticates the caller.

## Handle a notification

Each OCN carries three things: the booking reference (Order), a Status Code
indicating what changed, and Order Details. The message follows the NDC 17.2
OrderChangeNotification schema.

**Always follow the notification with OrderRetrieve.** Air Canada's instruction
is explicit: "Sellers are expected to retrieve the affected Order to view the
latest version of the Order." Do not treat the OCN payload as the new state.

## Status codes

Involuntary (act on these — the traveller's itinerary moved):

| Code | Meaning |
| --- | --- |
| 1 | Flight number change |
| 2 | Flight time change |
| 3 | Flight cancelled |
| 4 | Flight delayed |
| 5 | Flight added |
| 21 | Other involuntary change |
| 25 | Aircraft changed |
| 26 | Date change |
| 27 | Cabin change |

Passenger response to a disruption (reconcile, usually no action):

| Code | Meaning |
| --- | --- |
| 22 | Accepted by customer |
| 23 | Alternative chosen by customer |
| 24 | Cancelled by customer |

Voluntary and ticketing (update your record):

| Code | Meaning |
| --- | --- |
| 28 | Voluntary flight change |
| 29 | Voluntary seat change |
| 31 | Voluntary cancellation - retain for future credit |
| 32 | Voluntary name change |
| 33 | PNR on hold - itinerary cancelled by expired ticketing time limit |
| 34 | PNR on hold - ancillary cancelled by expired ticketing time limit |
| 35 | Voluntary travel option change |

Codes 6–20 and 30 are not published by Air Canada. Treat an unknown code as an
unclassified change: retrieve the Order and diff it, do not guess.

## Operational cautions

- No retry policy, delivery guarantee, ordering guarantee or dead-letter
  behaviour is published. Assume at-most-once delivery and reconcile on a
  schedule with OrderRetrieve for orders you care about.
- Because there is no order-list operation, your own store of OrderIDs is the
  only index you have. See `conventions/air-canada-conventions.yml`.

Full surface: `asyncapi/air-canada-ocn-webhooks.yml`.
