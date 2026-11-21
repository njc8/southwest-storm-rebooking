# Storm Rebooking Assistant: Spec (Denver, November 2026)

An assistant that prepares each storm rebooking for the agent, who confirms it with one click. Every rule cites the Storm Travel Waiver Fare Rules Bulletin (clause number), the call logs (Call Log ID), or interview INT-ROSALIND-01 ([hh:mm:ss]).

## Problem and baseline

- A storm rebooking takes 14.4 min on average (55 fully timed calls in storm_rebooking_call_logs.csv).
- Three steps take 10.6 of those minutes: Find Open Flights 5.4, Check Waiver Rules 3.0, Reseat Party 2.2.
- Agents page through flights one at a time on the seat screen [00:01:09].
- Agents re-check eligibility reasons the bulletin does not contain, such as a purchase-date cutoff (CL-003, CL-019, CL-046) or a flight list (CL-023, CL-059).
- Split-party calls average 22.4 min against 13.1 for a plain rebook (6 calls, e.g. CL-004, CL-039).
- Averages exclude CL-022, CL-029, CL-038, CL-041 and CL-055 (blank or non-summing step times).

## What the assistant does

1. Reads `waiver_eligible` from cancelled_bookings_denver_storm.csv and shows it as the decision (1.3, 1.4).
2. Offers only flights with the same origin and destination airport pair as the booking (2.2, 4.3).
3. Offers only flights whose `flight_status` is not cancelled. Delayed flights are shown with a flag (2.3, [00:12:52]).
4. Offers only flights whose `departure_datetime` is at or after the booking's `earliest_acceptable_departure` (2.4, 4.3).
5. Offers only flights departing 18–23 Nov 2026 inclusive (2.1). This applies to ineligible bookings too until Revenue Management says otherwise, because 2.5 sends every out-of-window request to them (see open question 2).
6. Offers only flights with enough open seats for the whole party, or a split in which every flight meets rules 2–5 (3.5).
7. For eligible bookings, shows: no fare difference in either direction, `fare_paid_usd` carries over, no change fees (3.1, 3.2, 3.4).
8. Ignores fare class when the booking is eligible. A different `fare_class_open` on the new flight changes nothing (3.3).
9. Ignores loyalty tier for fare treatment (4.2).
10. For ineligible bookings, shows the fare-difference rule for the original `fare_class` (4.1).
11. When no flight seats the party together, shows two options side by side: the earliest flight with seats together first, the best split second ([00:16:36], 3.5).
12. For eligible bookings, drafts the note line `STORM WAIVER DEN NOV26` plus the original confirmation number (3.6).
13. For split parties, drafts that line on every rebooked booking and cross-links each booking number in the others' notes (3.6, [00:14:34]).
14. Omits the waiver note line on ineligible bookings (4.4).
15. Sends different-airport requests, travel outside 18–23 Nov, and requests made after 23 Nov to Revenue Management, with no flight offered (2.5, 4.3; e.g. CL-012).

## What the agent confirms

- Which option the customer accepts. Customers turn down valid flights ([00:12:52], CL-012).
- Any split, and who travels separately. An adult should fly separately ([00:12:52], CL-039).
- The cross-linked booking numbers on a split before saving ([00:14:34]).
- Any fare difference quoted to an ineligible customer (CL-013, CL-042).
- Any refund request, handled under standard fare rules (4.1, CL-042).
- Any promise outside the bulletin, such as a hotel voucher ([00:03:45]).
- The final rebooking. The agent clicks confirm on every one.

## What the assistant never does

- Books, cancels or changes a booking on its own.
- Re-assesses eligibility or grants an exception (1.4). The bulletin has no exception path ([00:07:05]).
- Offers a flight for a request that must go to Revenue Management (2.5, 4.3).
- Moves other passengers' seats to make room ([00:13:43]).

## How we measure success

- Average handle time against the 14.4 min baseline.
- Find Open Flights, Check Waiver Rules and Reseat Party times against 5.4, 3.0 and 2.2 min.
- Split-party handle time against 22.4 min.
- Rebookings that break bulletin rules 2.1–2.4, from an audit of confirmed rebookings.
- No targets are set. Targets are an open question.

## Open questions

1. Is the 18 Nov start date in 2.1 correct? Eligible calls on 15–17 Nov booked same-day flights (CL-001, CL-011, CL-031). Owner: Revenue Management (bulletin prepared by Yolanda Prescott).
2. Does the 18–23 Nov window apply to ineligible bookings? 4.3 lists only 2.2–2.4 for them, but 2.5 sends all out-of-window requests to Revenue Management.
3. Where do fare disputes on ineligible tickets go? Today they go to a supervisor (CL-013, CL-042). The bulletin names no route.
4. We need cancelled_bookings_denver_storm.csv and flight_seat_availability.csv. Ask Devin Bartosz.
5. The listed fields include no fare price for the new flight. The assistant cannot compute a fare-difference amount (4.1) until a price source is named.
6. The listed fields include no seat count or seat adjacency. Rule 6 and rule 11 need them.
7. The listed fields include no onward connection. Agents check connection times today ([00:00:20]).
8. The listed fields include no passenger age. The assistant cannot tell which traveler is an adult, so the agent chooses who flies separately.
9. What handle-time target should the assistant be held to?
