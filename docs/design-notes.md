# P03 design notes

## Why design before code

Low-level design decides how classes are created inside a program, what each class does, what data it stores, and how classes work together.

## Where the methods live

- `Cancel()` lives in `Reservation`, because cancelling changes that reservation's own status.
- `HasOverlap(room, stay)` lives in `ReservationStore`, because the store can see all reservations and check whether another confirmed booking already uses the room on those nights.

## Activity traces

Omar asks for R7 on 13–15 November, while Maya has 12–14. First diamond: is check-out after check-in? Yes. Second diamond: does a Confirmed booking share a night? Yes, the 13th. The request ends at “Unavailable · save nothing”.

Nora asks for R7 on 14–16 November, while Maya has 12–14. First diamond: is check-out after check-in? Yes. Second diamond: does a Confirmed booking share a night? No, Maya has the nights of the 12th and 13th and Nora needs the 14th and 15th. The request goes through “Save the reservation as Confirmed” and “Send the confirmation email” and ends at “Accepted + reservation id” (R102).

Omar asks for R8 on 15–15 November. First diamond: is check-out after check-in? No, check-out is the same day as check-in, so there is no night at all. The request ends at “InvalidDates · save nothing”; the second diamond is never reached.

## Prediction

I expect Nora's line to be “Accepted R102”. Her nights (14th and 15th) do not share a night with Maya's (12th and 13th), and Omar's request was refused, so only R101 is saved before her and the next id is R102.

Result after running: “Nora books R7 14-16 Nov: Accepted R102”, so the prediction was right.

## The race

Prediction: R7 will have 2 Confirmed reservations for 12–14 November, because both requests can check the store before either one saves.

Output of `python hotel_booking.py --race`:

```
Maya and Omar both book R7 for 12-14 Nov at the same moment...
Maya: Accepted R101
Omar: Accepted R102
Confirmed reservations for R7 on 12-14 Nov: 2
Double booking! Both requests checked before either one saved.
```

What happened: both Maya's and Omar's requests asked the store while it was still empty, both got “false” and both saved, so R7 now has two Confirmed bookings for the same nights and R3 is broken. The problem appears in `Reserve` between `store.HasOverlap(room, stay)` (the check) and `store.Save(reservation)` (the act): during that gap the other request checks too and acts on an answer that is already out of date.

# P04 design notes

## Refactoring

A refactoring changes how the code is organised (names, files, which class depends on which) but must not change what the app answers to any request.

Must stay the same:

- The tests from Step 2: all 6 still pass after every move.
- The demo output: the same nine lines as in P03.
- The race demo: still “Double booking!”, because refactoring does not fix bugs.

## What our code breaks

- BookingService asks for exactly ReservationStore (D: dependency inversion).
  In P07 we would have to edit BookingService just to use a database.
- BookingService asks for exactly EmailSender (D and O: dependency inversion, open/closed).
  To send SMS we would have to open BookingService and change its constructor and Reserve(), and in the tests we cannot plug in a sender that fails, so we cannot prove that a failed email keeps the booking.
- All nine classes live in one file, hotel_booking.py (S: single responsibility).
  Every change touches the same file and it is hard to find the class you need.

## Proof

- Tests: `python -m unittest -v` → 6 of 6 passed after the refactoring, and no test body changed (only the import and `new_service()`).
- Demo: `python main.py` → the same nine lines as P03 (checked line by line with `diff` against the P03 output).
- Race: `python main.py --race` → still double-booked: Maya Accepted R101, Omar Accepted R102, 2 Confirmed reservations for R7.

After Step 7: the R6 test failed first (RuntimeError: The mail server is down.), then passed after the change in Reserve; SmsNotificationSender was added without editing BookingService. `python -m unittest -v` → 8 of 8 passed.
