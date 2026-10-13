# Notes

Write-ups on how specific parts of my public projects work. Each one quotes the code it describes and links to the exact lines at a fixed commit, then lists what the tests cover and where the code is still thin.

- [The trial balance was zero and the invoice was written off twice](ledgerline-balanced-but-wrong.md)\
  Ledgerline's retry loop kept invoice copies across a call that wrote them off, so one invoice was written off twice while the trial balance stayed at zero. The journal from a run on the old code, the two invariants that compare the books with the invoices, the fix that re-reads each invoice under lock, and why the API cannot reach that state today.
- [Two tabs, one slot: letting PostgreSQL refuse the second booking](slotbook-exclusion-constraint.md)\
  Why Slotbook lets an exclusion constraint, not a check in Go, refuse the second of two simultaneous bookings. A read-then-insert version that gave one slot to 14 of 20 requests, what the second insert waits for, why a reschedule cancels before it inserts, and a deadlock inside PostgreSQL's own check that turned some losers' `409` into a `500` until the transaction was made to run again.
- [Berlin on 29 March: booking slots across a DST change without adding 24 hours](slotbook-dst-slots.md)\
  Dates walked on UTC midnights, and wall times resolved through the instants that read back as the same time. 23 JSON vectors hold Slotbook's Go generator and its TypeScript port to the same strings in Berlin, New York, Lord Howe, Santiago and Cairo.
- [The late ack was the small bug: fencing a Redis queue with the lease deadline](relay-lease-fencing.md)\
  Part 2 of the Relay queue story. Making the lease deadline a fencing token showed that a worker past its lease overwrote the new owner's delivery record; every write after an attempt now checks the token, and a test pins the one place where I first got it wrong.
- [My Redis queue claimed jobs atomically and still handed the same job to two workers](relay-queue-races.md)\
  Two races in Relay's queue: `MULTI` made the claim atomic but not the read it was based on. A test hook that forces both interleavings on every run, the Lua fix, and a third gap still pinned with a strict xfail.
- [Relay's delivery queue on two Redis sorted sets with a 120-second lease](relay-redis-lease-queue.md)\
  How a worker leases a job instead of popping it, why the attempt is saved before the ack, the intake gap from issue #1, and two claim races that only show up with several workers.
- [How FlowDesk rotates refresh tokens with a conditional UPDATE and a cross-tab lock](flowdesk-refresh-rotation.md)\
  Single-use refresh tokens with family revocation, the conditional `updateMany` that lets only one of two concurrent refreshes win, and the Web Locks fix for two tabs on one session.
- [How Zernolist prices an order on the server and accepts each Telegram Stars charge once](zernolist-stars-payments.md)\
  Prices in kopecks from the catalog, the checks that run at pre-checkout and again on payment, and why a repeated `successful_payment` is not refunded.
- [How DocMind sends sources before the first token and links [n] markers mid-stream](docmind-sources-before-tokens.md)\
  The SSE event order, how the client turns `[n]` into a button while the text is still streaming, and what happens to a `[7]` when there are five sources.
- [How Pulse counts unique visitors with a daily salt and a sparse-to-dense HyperLogLog](pulse-hyperloglog.md)\
  A visitor id without cookies, a sketch that stays an exact set up to 512 hashes, where sketches merge, and the error measured at different sizes.

## По-русски

Разборы того, как устроены отдельные части моих открытых проектов. В каждом я цитирую код со ссылками на строки в конкретном коммите, а в конце пишу, что покрывают тесты и где код пока слабый.

- [Оборотная ведомость сходилась в ноль, а счёт списали дважды](ledgerline-balanced-but-wrong.ru.md)
- [Две вкладки, один слот: вторую запись отклоняет сам PostgreSQL](slotbook-exclusion-constraint.ru.md)
- [Берлин, 29 марта: слоты записи при переводе часов без прибавления 24 часов](slotbook-dst-slots.ru.md)
- [Поздний ack оказался мелочью: fencing token из срока аренды в очереди на Redis](relay-lease-fencing.ru.md)
- [Очередь на Redis атомарно забирала задачи и всё равно отдавала одну и ту же двум воркерам](relay-queue-races.ru.md)
- [Очередь доставок Relay на двух sorted set в Redis с арендой задачи на 120 секунд](relay-redis-lease-queue.ru.md)
- [Ротация refresh-токенов в FlowDesk через условный UPDATE и блокировку между вкладками](flowdesk-refresh-rotation.ru.md)
- [Как Zernolist считает цену заказа на сервере и принимает каждый платёж в Telegram Stars один раз](zernolist-stars-payments.ru.md)
- [Как DocMind отдаёт источники раньше первого токена и превращает [n] в ссылки на лету](docmind-sources-before-tokens.ru.md)
- [Как Pulse считает посетителей через суточную соль и HyperLogLog](pulse-hyperloglog.ru.md)
