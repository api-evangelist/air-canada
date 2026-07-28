---
name: Shop, price and book an Air Canada flight over NDC
description: >-
  Run the Air Canada NDC 17.2 prime booking flow end to end - AirShopping to
  OfferPrice to OrderCreate - honouring Air Canada's mandatory message order and
  identifier carry-forward rules.
api: air-canada:air-canada-ndc-airshopping-api
grounded_in:
  - https://ndc.aircanada.com/api/gettingstarted/apisetup
  - https://ndc.aircanada.com/api/gettingstarted/apisorchestration
  - https://ndc.aircanada.com/api/documentation/ndcapis
  - examples/UseCase1-AirshoppingRQ.xml
  - examples/UseCase2-OrderCreate-OrderCreateRQ.xml
operations: [AirShoppingRQ, OfferPriceRQ, OrderCreateRQ]
messages:
  - AirShoppingRQ / AirShoppingRS
  - OfferPriceRQ / OfferPriceRS
  - OrderCreateRQ / OrderViewRS
---

# Shop, price and book an Air Canada flight over NDC

Air Canada has no OpenAPI. The operations below are the IATA NDC 17.2 message
services Air Canada documents by name, at the endpoints Air Canada publishes in
its API Setup page. Do not invent messages that are not in this list.

## Before you start

- You need an Air Canada-issued `apikey`, a `SellerID`, and — for agency flows —
  your accredited 8-digit IATA number and the `AgencyID` Air Canada assigned you.
  There is no self-service signup; see `authentication/air-canada-authentication.yml`.
- Work in GOLD first: base `https://gold-ndcpartners.aircanada.com/`. Air Canada
  publishes test IATA numbers, test cards and test loyalty accounts in
  `sandbox/air-canada-sandbox.yml`. In production, pass your accredited IATA
  codes only — Air Canada states "Do not use any IATA codes that are used in examples."

## Transport

Every call is an HTTPS POST of a SOAP 1.1 envelope. The IATA message goes inside
`NDCMSG_Body/NDCMSG_Payload` in a CDATA section.

```
POST https://ndcpartners.aircanada.com/ipg-gw/ndc/17.2/v1/AirShopping
apikey: <your key>
Content-Type: application/xml
orc-debug: true
```

Envelope header values are fixed: `SchemaType: NDC`, `SchemaVersion: YY.2017.2`,
`Recipient/Address/Company: AC`. Copy the exact shape from
`examples/UseCase1-AirshoppingRQ.xml`.

## Steps

1. **AirShopping** — POST `ipg-gw/ndc/17.2/v1/AirShopping`. Supply
   `CoreQuery/OriginDestinations` (AirportCode + Date) and a `PassengerList` with
   a `PTC` per passenger. Air Canada prices one-way: a round-trip search returns
   a separate Offer for outbound and inbound. Offers come back sorted by
   ascending fare attribute, then by price within each fare.
   Attach any eligibility data here — corporate account code, Aeroplan
   `LoyaltyProgramAccount`, promo code, fare class. Air Canada's rule, verbatim:
   "For a specific account code, frequent flyer information, promotional code, or
   fare class you must carry this information from AirShopping through OrderCreate."

2. **(optional) SeatAvailability and ServiceList** — POST
   `ipg-gw/ndc/17.2/v1/SeatAvailability` and `ipg-gw/ndc/17.2/v1/ServiceList`
   against the flight OfferID. One Offer per SeatAvailabilityRQ: to seat both
   directions you must send two requests.

3. **OfferPrice** — POST `ipg-gw/ndc/17.2/v1/OfferPrice` with the flight OfferIDs
   from AirShoppingRS, plus seat OfferIDs from SeatAvailabilityRS and ancillary
   OfferIDs from ServiceListRS if you are quoting flights + seats + ancillaries.
   **This step is mandatory before OrderCreate.**

4. **OrderCreate** — POST `aps-gw/ndc/17.2/v1/OrderCreate` (note: the payment
   gateway, not `ipg-gw`). Send flight OfferIDs from OfferPriceRS, seat OfferIDs
   from SeatAvailabilityRS, ancillary OfferIDs from ServiceListRS, passenger
   detail and form of payment. A successful OrderViewRS returns the airline
   `OrderID` (e.g. `ORDER-c98c-4759-a20b`) and the Air Canada record locator.

5. **Persist the OrderID immediately.** There is no list, search or export
   operation. OrderRetrieve requires a known OrderID plus Owner. If you lose the
   id, you cannot enumerate what you booked.

## Error handling

- Errors arrive at HTTP 200 inside the `Errors` element with a numeric `Code`.
  Only transport failures use HTTP status; a missing `apikey` is HTTP 400.
- `89975` — OfficeID/credential validation failed. Check IATA number and AgencyID.
- `99970` / `99972` — no fare offer found, or the fare GUID is invalid/expired.
  Re-shop; do not retry the same ids.
- `99957` / `99960` — the offer is locked. Re-shop.
- `199994` / `199999` — CompanyShortName not supported / IATA not allowed. This is
  an accreditation problem, not a request problem.
- The `89987`–`89993` warnings are documented by Air Canada as ignorable.
- Full catalogue: `errors/air-canada-error-codes.yml`. Card declines:
  `errors/air-canada-decline-codes.yml`.

## Price change at OrderCreate

If OrderViewRS reports the quoted price is no longer available, re-run
OfferPriceRQ and resubmit OrderCreateRQ with the updated flight OfferIDs and
OfferItemIDs. Seat and ancillary OfferIDs still come from SeatAvailabilityRS and
ServiceListRS. **There is no idempotency key** — a retry with the same payload is
a new attempt, so always re-price rather than blindly retrying.

## Tracing

Set `orc-debug: true` on every request and capture the `orc-transaction-id`
response header. Air Canada's support desk requires that value in the
Transaction ID/Trace field of a ticket.
