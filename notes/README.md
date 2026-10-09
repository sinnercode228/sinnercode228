# Notes

Write-ups on how specific parts of my public projects work. Each one quotes the code it describes and links to the exact lines at a fixed commit, then lists what the tests cover and where the code is still thin.

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

- [Очередь доставок Relay на двух sorted set в Redis с арендой задачи на 120 секунд](relay-redis-lease-queue.ru.md)
- [Ротация refresh-токенов в FlowDesk через условный UPDATE и блокировку между вкладками](flowdesk-refresh-rotation.ru.md)
- [Как Zernolist считает цену заказа на сервере и принимает каждый платёж в Telegram Stars один раз](zernolist-stars-payments.ru.md)
- [Как DocMind отдаёт источники раньше первого токена и превращает [n] в ссылки на лету](docmind-sources-before-tokens.ru.md)
- [Как Pulse считает посетителей через суточную соль и HyperLogLog](pulse-hyperloglog.ru.md)
