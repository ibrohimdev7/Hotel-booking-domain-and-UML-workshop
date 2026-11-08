# L04 notes

1. 3.33 searches per second (200 a minute): 1,440 visitors × 20 searches ÷ 86,400 seconds × a peak factor of 10.
2. Each instance would have its own list, so two instances could both accept R7 for the same nights (even both as R101); one shared database lets every instance see every booking.
3. A stale result: after Maya books R7, a cached search can still show R7 as free for up to 30 seconds; booking never reads the cache, it always checks PostgreSQL (and every booking raises the cache version).
