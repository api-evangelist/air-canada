# Air Canada (air-canada)

Air Canada is Canada's flag carrier and largest airline, headquartered in Montreal, operating scheduled passenger service under the AC designator along with Air Canada Rouge, Air Canada Express and the Aeroplan loyalty programme. Its home market is Canada, where it forms a duopoly with WestJet. Air Canada sits at the supply end of the travel distribution chain: it is the sole source of its own seat inventory, reached either through the legacy GDS EDIFACT channel (Amadeus, Sabre, Travelport), through certified NDC aggregators, or directly through its own NDC API. Its API posture is unusual for a carrier in that the distribution surface is genuinely well documented in public: the ndc.aircanada.com developer portal publishes complete IATA NDC 17.2 (EDIST) message documentation, request/response element tables, error catalogues, use cases and downloadable sample XML with no login. What is not public is access itself — production credentials require a commercial agreement with Air Canada's distribution team, accredited IATA/ARC codes passed on every request, and passing Air Canada display certification test cases. There is no consumer API (no public flight status, booking or Aeroplan endpoint), no OpenAPI or machine-readable contract, no bulk export operation, and Air Canada publishes that it may revoke NDC programme access at its sole discretion. Public docs, gated access, no exit path.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/air-canada/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/air-canada/refs/heads/main/apis.yml)

## Tags

- Travel
- Canada
- Aviation
- Airline
- NDC
- Distribution
- Booking
- Airlines
- Loyalty

## Timestamps

- **Created:** 2026-07-28
- **Modified:** 2026-07-28

## Two Surfaces

Air Canada has two distinct API surfaces and they should not be conflated.

- **Consumer surface** — none published. `developer.aircanada.com`, `developers.aircanada.com`, `docs.aircanada.com`, `developer.aeroplan.com` and `api.aeroplan.com` do not resolve. `www.aircanada.com/developers`, `/api`, `/openapi.json` and `/swagger.json` all return 404. Flight status, check-in, booking and Aeroplan are web and app only.
- **Distribution surface** — documented in public at [ndc.aircanada.com](https://ndc.aircanada.com/), gated in practice. Ten IATA NDC 17.2 (EDIST) message services, XML over SOAP, reached through the `ndcpartners.aircanada.com` gateway once you are certified.

## APIs

### Air Canada NDC AirShopping API

IATA NDC 17.2 AirShopping message pair (AirShoppingRQ / AirShoppingRS). Shops one-way, round-trip and North America multicity itineraries and returns branded fare-family offers with per-passenger price breakdowns, calendar offers and flight details. Requires agent IATA number and Agency ID for agency flows, or Point of Sale for direct consumer flows.

- **Human URL:** [https://ndc.aircanada.com/api/documentation/airshopping](https://ndc.aircanada.com/api/documentation/airshopping)
- **Base URL:** `https://ndcpartners.aircanada.com`

### Air Canada NDC OfferPrice API

IATA NDC 17.2 OfferPrice message pair (OfferPriceRQ / OfferPriceRS). Returns detailed and comprehensive pricing for a selected offer, including base fare, tax breakdown, surcharges and fare rules.

- **Human URL:** [https://ndc.aircanada.com/api/documentation/offerprice](https://ndc.aircanada.com/api/documentation/offerprice)
- **Base URL:** `https://ndcpartners.aircanada.com`

### Air Canada NDC ServiceList API

IATA NDC 17.2 ServiceList message pair (ServiceListRQ / ServiceListRS). Returns optional and ancillary services purchasable against an offer or order, such as Maple Leaf Lounge access and Air Canada Bistro meal vouchers.

- **Human URL:** [https://ndc.aircanada.com/api/documentation/servicelist](https://ndc.aircanada.com/api/documentation/servicelist)
- **Base URL:** `https://ndcpartners.aircanada.com`

### Air Canada NDC SeatAvailability API

IATA NDC 17.2 SeatAvailability message pair (SeatAvailabilityRQ / SeatAvailabilityRS). Returns seat maps with advance and preferred seat pricing, during a booking flow against an OfferID or after an Order has been created against an OrderID.

- **Human URL:** [https://ndc.aircanada.com/api/documentation/seatavailability](https://ndc.aircanada.com/api/documentation/seatavailability)
- **Base URL:** `https://ndcpartners.aircanada.com`

### Air Canada NDC OrderCreate API

IATA NDC 17.2 OrderCreate message pair (OrderCreateRQ / OrderViewRS). Creates an actual reservation and returns an airline-assigned OrderID plus an Air Canada record locator and associated reservation details.

- **Human URL:** [https://ndc.aircanada.com/api/documentation/ordercreate](https://ndc.aircanada.com/api/documentation/ordercreate)
- **Base URL:** `https://ndcpartners.aircanada.com`

### Air Canada NDC OrderRetrieve API

IATA NDC 17.2 OrderRetrieve message pair (OrderRetrieveRQ / OrderViewRS). Retrieves a single Order by mandatory OrderID and Owner. Air Canada documents that retrieving a cancelled Order, a GDS booking, a redemption booking, or the modification history of an Order is not supported.

- **Human URL:** [https://ndc.aircanada.com/api/documentation/orderretrieve](https://ndc.aircanada.com/api/documentation/orderretrieve)
- **Base URL:** `https://ndcpartners.aircanada.com`

### Air Canada NDC OrderChange API

IATA NDC 17.2 OrderChange message pair (OrderChangeRQ / OrderViewRS). Applies servicing changes to an existing Order, including seat and ancillary servicing with or without itinerary change, passenger servicing and frequent-flyer updates.

- **Human URL:** [https://ndc.aircanada.com/api/documentation/orderchange](https://ndc.aircanada.com/api/documentation/orderchange)
- **Base URL:** `https://ndcpartners.aircanada.com`

### Air Canada NDC OrderReshop API

IATA NDC 17.2 OrderReshop message pair (OrderReshopRQ / OrderReshopRS). Reshops an existing Order to produce change offers, including partially flown itineraries and origin/destination replacement.

- **Human URL:** [https://ndc.aircanada.com/api/documentation/orderreshop](https://ndc.aircanada.com/api/documentation/orderreshop)
- **Base URL:** `https://ndcpartners.aircanada.com`

### Air Canada NDC OrderCancel API

IATA NDC 17.2 OrderCancel message pair (OrderCancelRQ / OrderCancelRS). Cancels an existing Order held by Air Canada, referenced by OrderID and Owner.

- **Human URL:** [https://ndc.aircanada.com/api/documentation/ordercancel](https://ndc.aircanada.com/api/documentation/ordercancel)
- **Base URL:** `https://ndcpartners.aircanada.com`

### Air Canada NDC OrderChangeNotification API

IATA NDC 17.2 OrderChangeNotif message. Air Canada-initiated notification of changes made to an Order outside the seller's own request, such as schedule change or involuntary disruption handling.

- **Human URL:** [https://ndc.aircanada.com/api/documentation/orderchangenotification](https://ndc.aircanada.com/api/documentation/orderchangenotification)
- **Base URL:** `https://ndcpartners.aircanada.com`

## Switching Cost

Recorded in full in [review.yml](review.yml). Summary:

| Dimension | Finding |
| --- | --- |
| Interface shape | `standard-plus-proprietary` — IATA NDC 17.2 / EDIST, extended by an Air Canada schema fork (`NDC2017.2-schemas-edist-ACNDC`) |
| Second source | `no-alternative` — aggregators and GDSs are interchangeable pipes to a single supplier |
| Exit path | `no-export-published` — no OrderList, no bulk export; OrderRetrieve needs a known OrderID, one at a time |
| Identifier portability | IATA airline/airport/agency codes and PADIS code lists are portable; OrderID, OfferID and the Air Canada record locator are not |
| Contractual lock-in | Published display requirements: audits, 30-day unilateral change notice, revocation at sole discretion |
| Access gate | `commercial-agreement`, plus accredited IATA/ARC codes and display certification |
| Distribution model | `gds-intermediated`, with a real `ndc-direct` option |
| NDC posture | IATA NDC 17.2 published and documented; Air Canada claims NDC@Scale certification on its own site |

## Common Properties

- [Website](https://www.aircanada.com/)
- [Developer Portal](https://ndc.aircanada.com/)
- [Documentation](https://ndc.aircanada.com/api/documentation/ndcapis)
- [Getting Started](https://ndc.aircanada.com/api/gettingstarted/gettingstarted)
- [Sign Up](https://ndc.aircanada.com/seller-registration-form)
- [Connection Options](https://ndc.aircanada.com/ndc-program/connection-options)
- [Sandbox](https://gold-ndc-sandbox.aircanada.com/login)
- [Support](https://ndc.aircanada.com/support)
- [FAQ](https://ndc.aircanada.com/support/faq)
- [Status](https://ndc.aircanada.com/support/statusMonitoring)
- [Release Notes](https://ndc.aircanada.com/api/releasenotes/latestrelease)
- [Roadmap](https://ndc.aircanada.com/api/roadmap)
- [NDC Display Requirements (PDF)](https://www.aircanada.com/content/dam/aircanada/portal/documents/PDF/ndc-displayRequirements-en.pdf)
- [Terms of Use](https://www.aircanada.com/ca/en/aco/home/legal/terms-of-use.html)
- [Privacy Policy](https://www.aircanada.com/ca/en/aco/home/legal/privacy-policy.html)
- [LinkedIn](https://www.linkedin.com/company/air-canada)

## Maintainers

- Kin Lane — kin@apievangelist.com
