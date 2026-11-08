# Quality scenarios

## Stakeholders

| Who | What they care about |
|---|---|
| Guest | Finds a free room quickly; never loses a booking |
| Receptionist | Never promises one room twice; can cancel at the desk |
| Hotel manager (domain expert) | Knows the rules; wants deposits and SMS soon |
| Development team | Can change one area without breaking the others |
| Email/SMS and payment providers | Outside systems that can be slow or down |

## S3 · Availability: a failed message keeps the booking

| Part | S3 |
|---|---|
| Source | The email provider |
| Stimulus | It stops answering |
| Environment | For 10 minutes, during normal bookings |
| Artifact | BookingService.Reserve and the notification sender |
| Response | Every valid booking is still Accepted and saved; the failed message is reported |
| Measure | 0 lost bookings; every failed message is counted |
| Checked in | P04 (R6 test, done) and P10 |

## S1 · Performance

| Part | S1 |
|---|---|
| Source | 200 guests |
| Stimulus | They search free rooms for 12–14 Nov |
| Environment | Within one minute |
| Artifact | The free-room search in BookingService |
| Response | Correct free rooms |
| Measure | p95 response ≤ 1 second |
| Checked in | P08 (cache), P23 (load test) |

## S2 · Data integrity

| Part | S2 |
|---|---|
| Source | 100 pairs of guests |
| Stimulus | Both guests of a pair book the same room and nights |
| Environment | At the same moment |
| Artifact | BookingService.Reserve and the reservation store |
| Response | Exactly one Confirmed reservation per pair |
| Measure | 0 double bookings |
| Checked in | P09 |

## S4 · Security

| Part | S4 |
|---|---|
| Source | A guest |
| Stimulus | Tries to read another guest's reservation |
| Environment | 100 times |
| Artifact | Viewing a booking by its id |
| Response | Another guest's reservation is never shown |
| Measure | 0 reservations shown to the wrong guest |
| Checked in | P21 |

## S5 · Modifiability

| Part | S5 |
|---|---|
| Source | The hotel manager |
| Stimulus | The hotel adds SMS confirmations |
| Environment | While the app is being developed |
| Artifact | BookingService and the notification senders |
| Response | BookingService is not edited |
| Measure | At most 2 files change |
| Checked in | P04 (SMS sender, done) |

## S6 · Sustainability

| Part | S6 |
|---|---|
| Source | Guests |
| Stimulus | The same 1,000 searches |
| Environment | Before and after a change |
| Artifact | The free-room search |
| Response | The same correct results with less energy |
| Measure | At least 20% fewer watt-hours per 1,000 completed searches |
| Checked in | P08, P24 |
