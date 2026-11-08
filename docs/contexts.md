# Hotel contexts

## Words

| Word | Meaning here | Name in our code |
|---|---|---|
| Stay | Check-in to check-out, half-open: 12–14 Nov = the nights of the 12th and 13th | DateRange |
| Reservation | Guest + room + stay + status, with an id such as R101 | Reservation |
| Confirmed | The room is promised to this guest for these nights | ReservationStatus.CONFIRMED |
| Guest | The person who stays in the room, with a name and an email: Maya, maya@example.com | Guest |
| Room | A room the hotel sells, such as R7 or R8 | Room (number, type) |
| Room type | The kind of room: Standard (R7) or Deluxe (R8) | Room.type |
| Nightly price | What one night in a room costs; Deluxe costs more than Standard | not in code yet |
| Out of service | The room is not offered to guests, for example R8 during repairs next week | not in code yet |
| Cancelled | The reservation is withdrawn and no longer blocks its nights | ReservationStatus.CANCELLED |
| Book | Ask for a free room for a stay and get a reservation id: Maya books R7 for 12–14 Nov → R101 | BookingService.reserve(guest, room, stay) |
| Cancel | Withdraw a reservation; it is kept as Cancelled: Maya cancels R101 | BookingService.cancel(reservation_id) |
| Free rooms | The rooms that no Confirmed reservation blocks for a stay: 12–14 Nov → R7, R8 | BookingService.find_free_rooms(stay) |
| Deposit | 30% of the stay's price, taken when the guest books | not in code yet |
| Paid | The deposit was taken | not in code yet |
| Declined | The deposit could not be taken, for example the card was refused | not in code yet |
| Confirmation message | The email or SMS that tells the guest the booking is accepted: “R101 is confirmed” | NotificationSender.send_confirmation(reservation) |
| Sent | The message reached the guest | not in code yet |
| Failed | The message could not be sent, for example the mail server is down | not in code yet |

## Rooms

Job: knows which rooms the hotel sells.

| Word | Meaning here |
|---|---|
| Room | A room the hotel sells, such as R7 or R8 |
| Room type | The kind of room: Standard (R7) or Deluxe (R8) |
| Nightly price | What one night in a room costs; Deluxe costs more than Standard |
| Out of service | The room is not offered to guests, for example R8 during repairs next week |

Rules this context owns:
- A room that is out of service is not offered to guests.
- The nightly price depends on the room type (Deluxe costs more than Standard).

Needs from other contexts: nothing.
Tells other contexts: nothing.

## Reservations

Job: decides whether a guest gets a room for some nights, and keeps every reservation.

| Word | Meaning here |
|---|---|
| Stay | Check-in to check-out, half-open: 12–14 Nov = the nights of the 12th and 13th |
| Reservation | Guest + room number + stay + status, with an id such as R101 |
| Confirmed | The room is promised to this guest for these nights |
| Cancelled | The promise is withdrawn; the nights are free again |
| Guest | The person who stays in the room, with a name and an email: Maya, maya@example.com |
| Book | Ask for a free room for a stay and get a reservation id: Maya books R7 for 12–14 Nov → R101 |
| Cancel | Withdraw a reservation; it is kept as Cancelled: Maya cancels R101 |
| Free rooms | The rooms that no Confirmed reservation blocks for a stay: 12–14 Nov → R7, R8 |

Rules this context owns:
- A room never has two Confirmed reservations that share a night.
- Check-out must be after check-in.
- Cancelling a reservation frees its nights for new bookings.

Needs from other contexts: the list of rooms (Rooms).
Tells other contexts: ReservationConfirmed (Notifications).

## Payments

Job: takes deposits and records whether they were paid.

| Word | Meaning here |
|---|---|
| Deposit | 30% of the stay's price, taken when the guest books |
| Paid | The deposit was taken |
| Declined | The deposit could not be taken, for example the card was refused |

Rules this context owns:
- The deposit is 30% of the stay's price.
- Only a Paid deposit can be refunded.

Needs from other contexts: nothing.
Tells other contexts: nothing.

## Notifications

Job: tells guests what happened, by email or SMS.

| Word | Meaning here |
|---|---|
| Confirmation message | The email or SMS that tells the guest the booking is accepted: “R101 is confirmed” |
| Sent | The message reached the guest |
| Failed | The message could not be sent, for example the mail server is down |

Rules this context owns:
- A failed message is recorded as Failed and never changes the reservation.

Needs from other contexts: nothing.
Tells other contexts: nothing.

## Same word, different meaning

| Word | Rooms | Reservations | Payments | Notifications |
|---|---|---|---|---|
| Status | In service / out of service | Confirmed / Cancelled | Paid / Declined | Sent / Failed |
| Room | R8 with its type, nightly price and whether it is in service | Only the room number, R8, written on a reservation | Not used | The text “Room R8” in the message |
| Cancel | Take a room off sale | Withdraw a reservation, so its nights are free again | Refund a deposit | Not used |

If Notifications could change a reservation's status to Failed, a failed SMS would overwrite Maya's Confirmed R101 and Maya would lose R7 because of a message, which breaks rule R6.

## Rule owners

| Card | Rule | Owner | Why this owner |
|---|---|---|---|
| 1 | A room never has two Confirmed reservations that share a night. | Reservations | It keeps every reservation of the room, so it can check them all together. |
| 2 | Check-out must be after check-in. | Reservations | The stay is its word, and it refuses wrong dates before anything else. |
| 3 | Cancelling a reservation frees its nights for new bookings. | Reservations | It keeps each reservation's status, and only Confirmed reservations block a room. |
| 4 | A room that is out of service is not offered to guests. | Rooms | It knows which rooms are on sale and which are out of service. |
| 5 | The nightly price depends on the room type (Deluxe costs more than Standard). | Rooms | It keeps each room's type and nightly price. |
| 6 | The deposit is 30% of the stay’s price. | Payments | Money rules belong to Payments; it asks Rooms for the nightly price. |
| 7 | Only a Paid deposit can be refunded. | Payments | It keeps each deposit's status, so it knows which deposits are Paid. |
| 8 | A failed message is recorded as Failed and never changes the reservation. | Notifications | It keeps the status of each message, so a failure stays there and never reaches the reservation (rule R6). |

## Between contexts

### Reservations → Notifications: ReservationConfirmed (event)

Sent when: a reservation is saved as Confirmed.

Fields:
- reservationId: R101
- guestEmail: maya@example.com
- roomNumber: R7
- checkIn: 2026-11-12
- checkOut: 2026-11-14

Reservations does not wait for an answer. A failed message never changes R101 (rule R6).

Never crosses: Reservations’ list of reservations. Notifications never reads it.

### Reservations → Rooms: which rooms exist? (question)

Sent when: Reservations searches free rooms or books a room.

What Reservations sends: only the question; no fields.

The answer and its fields: each room's number and type.
- number: R7, type: Standard
- number: R8, type: Deluxe

Never crosses: Reservations’ list of reservations. Rooms never reads it.

### Reservations → Payments: take a deposit (question, from P17)

Sent when: a guest books a room (from P17).

What Reservations sends:
- reservationId: R101
- amount: the deposit, 30% of the stay's price

The answer and its fields:
- status: Paid or Declined

Never crosses: Reservations’ list of reservations and the booking rule. Payments never decides whether a room is free.
