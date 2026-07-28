---
name: Cancel an Air Canada NDC order and choose the refund method
description: >-
  Cancel an existing Air Canada Order correctly - check eligibility on
  OrderRetrieve, call OrderReshop to see the cancellation methods and amounts,
  then OrderCancel with an explicit method so the booking is refunded rather than
  silently retained.
api: air-canada:air-canada-ndc-ordercancel-api
grounded_in:
  - https://ndc.aircanada.com/api/gettingstarted/apisorchestration
  - https://ndc.aircanada.com/api/documentation/ordercancel
  - https://ndc.aircanada.com/api/documentation/orderretrieve
  - examples/UseCase1-OrderCancel-OrderCancelRQ.xml
  - examples/Usecase1-OrderCancel-OrderCancelRS.xml
operations: [OrderRetrieveRQ, OrderReshopRQ, OrderCancelRQ]
messages:
  - OrderRetrieveRQ / OrderViewRS
  - OrderReshopRQ / OrderReshopRS
  - OrderCancelRQ / OrderCancelRS
---

# Cancel an Air Canada NDC order and choose the refund method

Cancelling on Air Canada NDC 17.2 is **two calls, in order**. Getting this wrong
costs the traveller money: Air Canada documents that if no cancellation method is
explicitly selected, OrderCancel retains the value for future use instead of
refunding it.

## Steps

1. **OrderRetrieve** — POST `ipg-gw/ndc/17.2/v1/OrderRetrieve` with the mandatory
   `OrderID` and `Owner` (`AC`). The Order is eligible for cancellation only if
   OrderViewRS contains a `FreeFormInstruction` with the remark
   `MODIFICATIONS ALLOWED = CL : CANCEL_PNR`. Check that before going further.

2. **OrderReshop** — POST `ipg-gw/ndc/17.2/v1/OrderReshop`. Air Canada's own
   words: this "provides details about cancellation like forfeit and retains,
   refund, etc." Read the eligible methods and their amounts from OrderReshopRS.

3. **OrderCancel** — POST `ipg-gw/ndc/17.2/v1/OrderCancel` with the OrderID, the
   Owner, and the cancellation method you selected in step 2. Air Canada's
   guidance, verbatim: "if no method is explicitly selected, OrderCancel will not
   refund and instead Retain for future use. AC recommends to consume OrderCancel
   always after OrderReshop."

## What you cannot do

- You cannot retrieve a cancelled Order afterwards — Air Canada documents that
  "Retrieving a Cancelled Order is not supported." Capture the OrderCancelRS
  before you lose it.
- You cannot retrieve the Order's modification history ("Air Canada does not
  support Order versioning yet").
- You cannot filter OrderRetrieve to sections of the Order.
- GDS bookings and redemption bookings cannot be retrieved through this API at all.

## Error handling

Cancellation and refund errors are largely Amadeus host errors surfaced through
the NDC `Errors` element:

- `1929` INVALID RECORD LOCATOR, `1931` NO MATCH FOR RECORD LOCATOR, `1959` NEED PNR
- `1914` OFFICE RESTRICTED, `1533` INVALID OFFICE IDENTIFICATION CODE
- `99990` Unable to determine 1A record locator, `99983` NO MATCH FOR RECORD LOCATOR - NAME
- `2241` ACCESS DENIED, `25677` ATC REFUND NOT AUTHORIZED, `27520` TRANSACTION NOT
  ALLOWED/REFUND MANUALLY — these mean the refund cannot be completed
  programmatically and needs the service desk.
- `2162` LINK DOWN - RETRY IN 2 MINUTES and `25715` FARE SERVER TEMPORARY DOWN -
  PLEASE RETRY IN 2MIN are the two errors Air Canada explicitly frames as retryable.

Full catalogue: `errors/air-canada-error-codes.yml`.

## Reconciliation

Cancellation can also arrive from Air Canada's side. If you are registered for
OrderChangeNotification, reason code `24` (Involuntary Change - Cancelled by
Customer) and `31` (Voluntary Cancellation - Retain for Future Credit) will be
pushed to your webhook. Always follow an OCN with an OrderRetrieve — the
notification is a change signal, not full state. See
`asyncapi/air-canada-ocn-webhooks.yml`.
