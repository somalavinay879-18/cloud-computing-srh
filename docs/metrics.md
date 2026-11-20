# Delay per lane: SHP02 Nordic Home

Source: `freight.bookings` joined to `freight.schedules`, October 2026, query in
`sessions/03-batch-streams-and-architecture-patterns/starter/delay_per_lane.sql`.
On time = arrived no more than 24 hours after planned arrival.
Reliability target: **75% on time on every lane**.

| Lane | Trade lane | Shipments | Avg delay (h) | On time | Target | Result |
|---|---|---|---|---|---|---|
| KRPUS → USLGB | Transpacific | 35 | 55.5 | 45.7% | 75% | Misses |
| NLRTM → USHOU | Transatlantic | 38 | 42.3 | 63.2% | 75% | Misses |
| DEHAM → USNYC | Transatlantic | 30 | 53.8 | 63.3% | 75% | Misses |
| SGSIN → BEANR | Asia-North Europe | 28 | 6.6 | 100.0% | 75% | Meets |

## Which lanes miss the target, and which carriers serve them?

Three of four lanes miss the 75% target; only SGSIN → BEANR meets it.

| Lane | Carrier | Shipments | On time | Avg delay (h) |
|---|---|---|---|---|
| KRPUS → USLGB | HRZ | 21 | 61.9% | 51.7 |
| KRPUS → USLGB | NWL | 9 | 0.0% | 74.7 |
| KRPUS → USLGB | KST | 3 | 66.7% | 44.0 |
| DEHAM → USNYC | HRZ | 16 | 50.0% | 61.5 |
| DEHAM → USNYC | KST | 6 | 50.0% | 99.0 |
| DEHAM → USNYC | NWL | 5 | 100.0% | 4.8 |
| NLRTM → USHOU | HRZ | 22 | 77.3% | 33.8 |
| NLRTM → USHOU | NWL | 6 | 66.7% | 28.0 |
| NLRTM → USHOU | ORS | 4 | 0.0% | 99.0 |

Carriers with three or fewer shipments on a lane are left out.

- **KRPUS → USLGB** is the worst lane: NWL delivered none of its 9 shipments on time, and HRZ, the main carrier, is at 62%.
- **DEHAM → USNYC**: HRZ and KST are both at 50%; NWL was on time on all 5.
- **NLRTM → USHOU**: HRZ alone would meet the target (77%); ORS (0 of 4) pulls the lane down.
- HRZ carries most of Nordic Home's volume, so its reliability decides whether the target is met.

28 of 159 Nordic Home bookings sail after October and have no schedule yet, so they are not counted.
