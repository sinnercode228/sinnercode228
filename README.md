<p>
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/banner-dark.png">
  <img src="assets/banner-light.png" alt="sinnercode (sinnercode228 on GitHub), open to contract work. Full-stack developer, commercial web development since 2019. 2 Python libraries on PyPI: aiogram-stars and aioamocrm. 7 of 8 projects have a live demo on GitHub Pages, with no backend. Pull request #1956 to aiogram, adding tasks_shutdown_timeout, is in review." width="100%">
</picture>
</p>

[Portfolio](https://sinnercode228.github.io/portfolio/?lang=en) · Telegram [@sinnercode](https://t.me/sinnercode) · [PyPI](https://pypi.org/user/sinnercode/) · [dev.to](https://dev.to/sinnercode228) · [По-русски](#по-русски)

I have done commercial web development since 2019. Now I build web products end to end and work on existing systems. The public projects below cover APIs, online booking and subscription billing, integrations, Telegram bots and Mini Apps, and LLM features; two Python libraries that grew out of them are on PyPI.

Open to contract work: a fixed-price project, hourly or monthly work, a seat on your team, or taking over an existing codebase.

## Featured

<table>
<tr>
<td width="50%" valign="top">
<a href="https://github.com/sinnercode228/aiogram-stars"><img src="assets/card-aiogram-stars.png" width="100%" alt="aiogram-stars: Telegram Stars payments for aiogram 3, pip install aiogram-stars, and the payment flow from pre_checkout_query to on_paid and refunds"></a><br>
<b><a href="https://github.com/sinnercode228/aiogram-stars">aiogram-stars</a></b> · <a href="https://pypi.org/project/aiogram-stars/">PyPI</a><br>
Telegram Stars payments for aiogram 3: calls <code>on_paid</code> once per charge and keeps refunds in your database until Telegram confirms them.
</td>
<td width="50%" valign="top">
<a href="https://github.com/sinnercode228/aioamocrm"><img src="assets/card-aioamocrm.png" width="100%" alt="aioamocrm: async Python client for the amoCRM API v4 and Kommo, pip install aioamocrm, and a token refresh followed by a POST that is not resent after a 429"></a><br>
<b><a href="https://github.com/sinnercode228/aioamocrm">aioamocrm</a></b> · <a href="https://pypi.org/project/aioamocrm/">PyPI</a><br>
Async Python client for the amoCRM API v4 and Kommo, with retries that never send a write twice unless you ask.
</td>
</tr>
<tr>
<td width="50%" valign="top">
<a href="https://github.com/sinnercode228/slotbook"><img src="assets/card-slotbook.png" width="100%" alt="Slotbook: booking API where PostgreSQL rules out double booking, with the staff day calendar"></a><br>
<b><a href="https://github.com/sinnercode228/slotbook">Slotbook</a></b> · <a href="https://sinnercode228.github.io/slotbook/">demo</a><br>
Booking backend in Go: two customers can never book the same specialist for overlapping time, and PostgreSQL enforces it, not a check in Go.
</td>
<td width="50%" valign="top">
<a href="https://github.com/sinnercode228/pulse-analytics"><img src="assets/card-pulse.png" width="100%" alt="Pulse: cookieless web analytics and uptime monitoring, with the traffic dashboard"></a><br>
<b><a href="https://github.com/sinnercode228/pulse-analytics">Pulse</a></b> · <a href="https://sinnercode228.github.io/pulse-analytics/">demo</a><br>
Cookieless web analytics and HTTP uptime monitoring in one dashboard, with a HyperLogLog visitor sketch in every rollup row.
</td>
</tr>
</table>

## Experience

**2025 – now · Senior Full-Stack Developer · Independent / Contract**\
Full-cycle web products and work on existing systems: architecture, dashboards and admin panels, APIs and integrations, optimizing queries, job queues and large imports, CI/CD, monitoring, backup and restore, code review, refactoring and releases.

**2023 – 2025 · Full-Stack Developer · Independent projects and contracts**\
Services with subscriptions, payments and automation: a SaaS for links and contacts, a Telegram sales platform with automatic delivery of digital goods, a website analytics service. I designed the API and database, built the UI, deployed and supported them. Next.js, TypeScript, Node.js, Python, PostgreSQL, Redis, Nginx.

**2021 – 2023 · Full-Stack Developer · Contract development**\
Internal web apps for sales departments and service companies: a CRM for handling requests, a document portal with approvals, supplier catalog sync with price and stock updates. React, TypeScript, Node.js, PostgreSQL, Redis, Docker.

**2019 – 2021 · Web Developer · Freelance**\
Websites and online stores for small businesses: an auto parts store with CSV import, a site for a chain of repair shops with a cost calculator and requests sent to a CRM, WordPress sites with custom themes. JavaScript, PHP, WordPress, MySQL.

The commercial code from this work is private, so the public repositories below are standalone projects built on made-up data.

## Libraries

Both are on PyPI at version 0.1.0, MIT licensed, for Python 3.10 or newer.

**[aiogram-stars](https://github.com/sinnercode228/aiogram-stars)** · Python, aiogram 3, SQLAlchemy, SQLite, PostgreSQL · [PyPI](https://pypi.org/project/aiogram-stars/) · [write-up](https://dev.to/sinnercode228/the-stars-were-charged-and-onpaid-never-ran-where-telegram-stars-payments-get-lost-in-aiogram-5m2) · `pip install aiogram-stars`\
Telegram Stars payments for aiogram 3, extracted from the payment code of [Zernolist](https://github.com/sinnercode228/tg-shop-miniapp): signed invoice payloads, a pre-checkout answer within Telegram's 10-second deadline, and a refund outbox in SQLite or PostgreSQL that retries `refundStarPayment` on errors. `on_paid` runs once per charge: the payment row is [inserted with `ON CONFLICT (charge_id) DO NOTHING`](https://github.com/sinnercode228/aiogram-stars/blob/d601bc348503130979eee6c2313de45265dbb23d/src/aiogram_stars/sqlalchemy.py#L364-L369), only the handler whose insert went through calls it, and a test feeds one update three times to each of two dispatchers on one database.

**[aioamocrm](https://github.com/sinnercode228/aioamocrm)** · Python, httpx, pydantic 2 · [PyPI](https://pypi.org/project/aioamocrm/) · `pip install aioamocrm`\
Async client for the amoCRM and Kommo API v4 that grew out of the amoCRM connector in [Relay](https://github.com/sinnercode228/integration-hub): OAuth2 with refresh-token rotation, requests spaced at 7 per second, pagination, typed models and webhook parsing. A request that got a 429 or a 5xx, or timed out after it was sent, is resent only if [its method is idempotent or the call passed `retry=True`](https://github.com/sinnercode228/aioamocrm/blob/a35f0c86ac0521a8397e90ae79e3fb32dad88caa/src/aioamocrm/_client.py#L264-L265), because amoCRM has no idempotency keys and a resent `POST` can create the lead twice. The tests run on mocked HTTP, with request and answer shapes checked against amoCRM's official PHP SDK, not a live account.

## Public projects

The demos run on GitHub Pages without a backend. The API is replaced by code in the browser.

**[Pulse](https://github.com/sinnercode228/pulse-analytics)** · TypeScript, Fastify, SQLite, React · [demo](https://sinnercode228.github.io/pulse-analytics/) · [write-up](notes/pulse-hyperloglog.md)\
Cookieless web analytics with uptime monitoring. Each rollup row stores a [HyperLogLog sketch](https://github.com/sinnercode228/pulse-analytics/blob/96645e153317f7bc35e5fc17bf4f76424d83c7c3/packages/core/src/hll.ts#L3-L7): an exact set up to 512 hashes, then 2048 registers. That is how unique visitors are merged across time buckets.

<a href="https://sinnercode228.github.io/pulse-analytics/"><img src="assets/pulse-live.webp" width="100%" alt="Pulse dashboard: the live feed, online counter and realtime bars updating"></a>
<sub>Recorded from the <a href="https://sinnercode228.github.io/pulse-analytics/">Pulse demo</a> on GitHub Pages. The traffic is synthetic.</sub>

**[Ledgerline](https://github.com/sinnercode228/ledgerline-billing)** · TypeScript, Fastify, PostgreSQL, React · [demo](https://sinnercode228.github.io/ledgerline-billing/) · [write-up](notes/ledgerline-balanced-but-wrong.md)\
Subscription billing with trials, proration, gapless invoice numbers, dunning and a double-entry ledger. A [constraint trigger deferred to COMMIT](https://github.com/sinnercode228/ledgerline-billing/blob/442e27c08854c39821ade0cbfe7ebe67bffd2c1c/server/migrations/002_ledger_guards.sql#L7-L50) rejects any journal transaction whose entries do not sum to zero, whatever code wrote it. The demo runs the same engine in the browser, with a test clock that moves a day, a week or a month at a time.

**[Slotbook](https://github.com/sinnercode228/slotbook)** · Go, PostgreSQL, React · [demo](https://sinnercode228.github.io/slotbook/) · [write-up](notes/slotbook-exclusion-constraint.md)\
Online booking for salons, with a booking widget and a staff day view. Double booking is ruled out by [one `EXCLUDE USING gist` constraint](https://github.com/sinnercode228/slotbook/blob/1c8318474319f6e61cc76a3bbc5b5911625d7c1e/internal/db/migrations/0001_schema.sql#L111-L113), not by a check in Go: when 20 goroutines hold the same slot, exactly one succeeds. 23 JSON vectors pin how slots behave across DST changes, and both the Go generator and its TypeScript port pass them.

**[Relay](https://github.com/sinnercode228/integration-hub)** · Python, FastAPI, Redis, SQLite, React · [demo](https://sinnercode228.github.io/integration-hub/) · [write-up](notes/relay-redis-lease-queue.md)\
Takes webhooks from Tilda, amoCRM and Bitrix24 and delivers them to Telegram, Google Sheets, amoCRM and email. The [Redis queue is built on sorted sets](https://github.com/sinnercode228/integration-hub/blob/b0ae35c14d753930be6c89155cdc75d31b861ff6/backend/src/relay/queue/redis.py#L29-L47): a worker claims a job with a lease (120 seconds by default) in one Lua script, and if the worker dies, the job goes back to the queue when the lease runs out.

**[DocMind](https://github.com/sinnercode228/docmind-rag)** · Python, FastAPI, pgvector, React · [demo](https://sinnercode228.github.io/docmind-rag/) · [write-up](notes/docmind-sources-before-tokens.md)\
Answers questions over PDF, DOCX, HTML files and web pages with citations, and has a Telegram bot. [Sources go out over SSE before the model is called](https://github.com/sinnercode228/docmind-rag/blob/6d8b1b24fd2b3a06cbc685e1b0702e8243269cc4/backend/src/docmind/rag/service.py#L110-L111), so each `[n]` is clickable as soon as it streams in. The demo has no LLM: it searches with BM25 in the browser.

<a href="https://sinnercode228.github.io/docmind-rag/"><img src="assets/docmind-answer.webp" width="100%" alt="DocMind demo: a question, the answer streaming with [1] and [2], and the matching source cards highlighted"></a>

**[Zernolist](https://github.com/sinnercode228/tg-shop-miniapp)** · React, aiogram 3, FastAPI, SQLAlchemy · [demo](https://sinnercode228.github.io/tg-shop-miniapp/) · [write-up](notes/zernolist-stars-payments.md)\
A coffee and tea shop inside Telegram, with payment in Stars or on receipt. The server [prices every order from the catalog](https://github.com/sinnercode228/tg-shop-miniapp/blob/54cb076af58b99b8e693ff301d43ea20a9dc1d91/bot/tgshop/services/orders.py#L76-L100) and ignores client prices: a test sends `price: 1` and still gets the catalog subtotal.

**[FlowDesk](https://github.com/sinnercode228/flowdesk-crm)** · Next.js, Fastify, Prisma, PostgreSQL · [demo](https://sinnercode228.github.io/flowdesk-crm/) · [write-up](notes/flowdesk-refresh-rotation.md)\
A CRM with a deals kanban and roles. Refresh tokens rotate, and a reused one revokes every token from that login. A [conditional `updateMany` inside a transaction](https://github.com/sinnercode228/flowdesk-crm/blob/de0a2c21755f9f59060d9d6d07f55e0b3b1cd42b/server/src/modules/auth/auth.service.ts#L44-L51) stops two concurrent refreshes from both getting a new pair.

**[n8n-home-ai-workflows](https://github.com/sinnercode228/n8n-home-ai-workflows)** · n8n, Ollama, Qdrant, Docker Compose\
Four workflows: meeting notes to Notion via Claude, a family calendar digest, document Q&A on local models, an error handler. Code-node logic is built from ES modules, and the [tests run the `jsCode` stored in the workflow JSON](https://github.com/sinnercode228/n8n-home-ai-workflows/blob/d314e04e913c38c894f52360271582a54ccf68e0/tests/harness.mjs#L51). No live demo, and I haven't run the workflows against live Anthropic, Notion, Google or Telegram accounts.

Smaller demos are in [portfolio](https://github.com/sinnercode228/portfolio): a landing page with a cost calculator, an async scraper, a lead-collection bot and an Excel dashboard.

## Notes

Write-ups on how specific parts of these projects work. Each one quotes the code, links to the lines at a pinned commit, then says what the tests cover and where the code is still thin. Index: [notes/](notes/).

- [The Stars were charged and on_paid never ran: where Telegram Stars payments get lost in aiogram](https://dev.to/sinnercode228/the-stars-were-charged-and-onpaid-never-ran-where-telegram-stars-payments-get-lost-in-aiogram-5m2) (dev.to): a `wait_for` that let the pre-checkout answer leave after 11 s, a catch-all router that took the payment, a shutdown that cut `on_paid` off, and a 429 that burnt an attempt of every refund
- [The trial balance was zero and the invoice was written off twice](notes/ledgerline-balanced-but-wrong.md): a retry loop working from stale invoice copies, and the invariants that catch what the trial balance cannot
- [Two tabs, one slot: letting PostgreSQL refuse the second booking](notes/slotbook-exclusion-constraint.md): a check in Go that gave one slot to 14 of 20 requests, and the retry that stopped a deadlock from turning a `409` into a `500`
- [Berlin on 29 March: booking slots across a DST change without adding 24 hours](notes/slotbook-dst-slots.md): dates walked on UTC midnights, and 23 vectors shared by Go and TypeScript
- [The late ack was the small bug: fencing a Redis queue with the lease deadline](notes/relay-lease-fencing.md): part 2, where the fencing token turns up a worse bug
- [My Redis queue claimed jobs atomically and still handed the same job to two workers](notes/relay-queue-races.md): two races, a test that forces them on every run, and the fix
- [Relay's delivery queue on two Redis sorted sets with a 120-second lease](notes/relay-redis-lease-queue.md)
- [How FlowDesk rotates refresh tokens with a conditional UPDATE and a cross-tab lock](notes/flowdesk-refresh-rotation.md)
- [How Zernolist prices an order on the server and accepts each Telegram Stars charge once](notes/zernolist-stars-payments.md)
- [How DocMind sends sources before the first token and links [n] markers mid-stream](notes/docmind-sources-before-tokens.md)
- [How Pulse counts unique visitors with a daily salt and a sparse-to-dense HyperLogLog](notes/pulse-hyperloglog.md)

## Open issues

_None right now._

## Recently fixed

Bugs I found in my own projects, each fixed with a test that failed first:

- DocMind: the prompt escaped a source's title but not its text, so a document containing `</source>` could close its block early and put the rest of the chunk outside the sources; source tags inside the text are now escaped and the rest is left as is ([issue #4](https://github.com/sinnercode228/docmind-rag/issues/4), [#5](https://github.com/sinnercode228/docmind-rag/pull/5))
- Relay: a Telegram message over 4096 characters was cut on the raw string, so in HTML mode the cut could split an entity or leave a tag open, Telegram answered 400 and the delivery went to the dead letters; the limit now counts only visible text, as Telegram does, and tags still open at the cut are closed ([issue #4](https://github.com/sinnercode228/integration-hub/issues/4), [#5](https://github.com/sinnercode228/integration-hub/pull/5))
- DocMind: a model that wrote citations as `[1, 2]` instead of `[1][2]` yielded an empty `cited` list and left every source card greyed out; the server regex and the client parser now split a bracket group into separate references ([issue #2](https://github.com/sinnercode228/docmind-rag/issues/2), [#3](https://github.com/sinnercode228/docmind-rag/pull/3))
- Zernolist: a `pending` result from `openInvoice` was collapsed into `payment=cancelled` on the redirect, so the Mini App told the user the payment had been cancelled although the Stars charge might still land; the status is now carried through and the pending case has its own note ([issue #5](https://github.com/sinnercode228/tg-shop-miniapp/issues/5), [#6](https://github.com/sinnercode228/tg-shop-miniapp/pull/6))
- Home AI flows: task owner resolved to the wrong attendee when two attendees shared a first name; the first-name fallback now only fires when exactly one attendee matches ([issue #3](https://github.com/sinnercode228/n8n-home-ai-workflows/issues/3), [#5](https://github.com/sinnercode228/n8n-home-ai-workflows/pull/5))
- Relay: a connection error or timeout on the amoCRM note, the request that follows the lead, made the worker retry the delivery and create the lead a second time; any failure of the note now goes into the delivery's result, and the note gives up before the attempt's timeout ([`fb4dbe9`](https://github.com/sinnercode228/integration-hub/commit/fb4dbe9c08d417cd35ccbdfa7bdc3edb0c5e068d))
- Slotbook: two writes on one period at the same moment could deadlock inside PostgreSQL's exclusion-constraint check, and the request whose transaction the deadlock detector aborted got `500`: a loser instead of `409 slot_taken`, a confirmation instead of going through; holds, blocks, reschedules and confirmations now run the transaction again on `40P01` ([`54ccd48`](https://github.com/sinnercode228/slotbook/commit/54ccd486199c9392846f2730a82e3a3e73a830b4), [`939022a`](https://github.com/sinnercode228/slotbook/commit/939022ae1a3c00362d064e0a11dee60eb2d4958d); see [the write-up](notes/slotbook-exclusion-constraint.md))
- Ledgerline: when a failed retry ended dunning inside the retry loop, every open invoice was written off, and the loop's stale copy of the next one reopened it, recorded another payment attempt and wrote it off a second time; the trial balance stayed at zero ([`b8aabf0`](https://github.com/sinnercode228/ledgerline-billing/commit/b8aabf0576976b7dade1b8158720c4b6c4cc7c99); see [the write-up](notes/ledgerline-balanced-but-wrong.md))
- Slotbook: the notification worker sent a claimed batch one row at a time under one lease, so with slow sends a row late in the batch could be reclaimed by another worker and sent by both; each row's lease is now renewed right before its send ([`a38902f`](https://github.com/sinnercode228/slotbook/commit/a38902fac3d147b686191bbc63534f8b2b3f0193))
- Relay: two claim races with several workers on one Redis, then a stale worker overwriting the new owner's record ([`b0ae35c`](https://github.com/sinnercode228/integration-hub/commit/b0ae35c14d753930be6c89155cdc75d31b861ff6), [`8c6ccb4`](https://github.com/sinnercode228/integration-hub/commit/8c6ccb42a436863df19aadec3e896cd45b4cf98f); see the two Relay notes above)
- Relay: an event stored but never queued when enqueue fails ([issue #1](https://github.com/sinnercode228/integration-hub/issues/1), [`c845ca0`](https://github.com/sinnercode228/integration-hub/commit/c845ca096591389e34603adba0a22f3bbac12368))
- DocMind: `docmind ask` finding nothing after `docmind ingest` with default settings, and `[0]` turning into a citation ([issue #1](https://github.com/sinnercode228/docmind-rag/issues/1), [`d89f9ea`](https://github.com/sinnercode228/docmind-rag/commit/d89f9ea1a8f19103ae485af111d1c3dec5baae39))
- Home AI flows: the doc-QA chunker dropping the start of a long paragraph with no whitespace, so the document was embedded from a fragment ([issue #1](https://github.com/sinnercode228/n8n-home-ai-workflows/issues/1), [#2](https://github.com/sinnercode228/n8n-home-ai-workflows/pull/2))
- Relay: a request longer than the lease could be sent twice; the lease is now renewed while the connector runs ([`c4be59c`](https://github.com/sinnercode228/integration-hub/commit/c4be59c91e59dc628734edbcdb2e5ec0c2d9f737))
- FlowDesk: a published fallback JWT secret accepted in production, a login limit bypassed with `X-Forwarded-For`, and refresh tokens that were never deleted ([`4770f75`](https://github.com/sinnercode228/flowdesk-crm/commit/4770f75891eb6648a4edc7d01a99ae70e1668454), [`af961e9`](https://github.com/sinnercode228/flowdesk-crm/commit/af961e9336ece99694ac0c72c1d998f90d0aa188))
- Pulse: live ranges dropping the current minute, hour or day, and uptime monitors probed every 5 minutes whatever their interval ([`b6a0fa4`](https://github.com/sinnercode228/pulse-analytics/commit/b6a0fa4275fb3f4ef1dbc1f1c269f8f071accc51), [`83ce266`](https://github.com/sinnercode228/pulse-analytics/commit/83ce266ede6cee82de3c1c36b6e304024d534732))
- Zernolist: two payments, or a payment and a cancel, racing on one order on SQLite ([`fa866de`](https://github.com/sinnercode228/tg-shop-miniapp/commit/fa866decb9ae35c63ae8a0a74d6b9f99d7ad615d))

## Contact

Telegram [@sinnercode](https://t.me/sinnercode) · [portfolio](https://sinnercode228.github.io/portfolio/?lang=en)

---

## По-русски

Коммерческая веб-разработка с 2019 года. Сейчас занимаюсь разработкой веб-продуктов полного цикла и развитием существующих систем. В открытых проектах выше — API, онлайн-запись и биллинг подписок, интеграции, Telegram-боты и Mini Apps, функции на LLM. Беру проекты с фиксированной ценой, почасовую и помесячную работу, могу войти в вашу команду или разобраться в чужом коде.

- **2025 — сейчас.** Senior Full-Stack Developer, independent / contract. Полный цикл разработки веб-продуктов и развитие существующих систем.
- **2023–2025.** Full-Stack Developer, независимые проекты и контракты. Сервисы с подписками, платёжными интеграциями и автоматизацией.
- **2021–2023.** Full-Stack Developer, контрактная разработка. Внутренние веб-приложения для отделов продаж и сервисных компаний.
- **2019–2021.** Web Developer, фриланс. Сайты и интернет-магазины для малого бизнеса.

Коммерческий код из этой работы закрыт, поэтому открытые репозитории выше — самостоятельные проекты на выдуманных данных. README у них на английском; у всех, кроме Relay, есть русская версия или короткий пересказ по-русски.

Две мои библиотеки на Python опубликованы на PyPI, обе пока в версии 0.1.0:

- [aiogram-stars](https://github.com/sinnercode228/aiogram-stars), [PyPI](https://pypi.org/project/aiogram-stars/): библиотека для оплаты в Telegram Stars в ботах на aiogram 3, которая подписывает payload счёта, успевает ответить на pre-checkout за десять секунд, вызывает `on_paid` один раз на платёж и повторяет неудавшиеся возвраты; я вынес её из платёжного кода Zernolist
- [aioamocrm](https://github.com/sinnercode228/aioamocrm), [PyPI](https://pypi.org/project/aioamocrm/): асинхронный клиент API v4 amoCRM и Kommo, который сам обновляет OAuth2-токены и сохраняет новый refresh-токен, отправляет не больше 7 запросов в секунду, а запрос на запись, который мог уже выполниться, без `retry=True` не повторяет; вырос из коннектора amoCRM в Relay; форматы запросов и ответов сверены с официальным PHP SDK amoCRM, но живого аккаунта за тестами пока нет

Ошибки, которые я нашёл у себя и исправил, каждую с тестом, который сначала падал, перечислены выше в разделе Recently fixed.

Разборы того, как устроены отдельные части проектов, с кодом и ссылками на строки:

- [Оборотная ведомость сходилась в ноль, а счёт списали дважды](notes/ledgerline-balanced-but-wrong.ru.md): цикл повторов на устаревших копиях счетов и инварианты, которые видят то, чего не видит оборотная ведомость
- [Две вкладки, один слот: вторую запись отклоняет сам PostgreSQL](notes/slotbook-exclusion-constraint.ru.md): проверка на Go, которая отдала один слот 14 запросам из 20, и повтор транзакции, после которого дедлок больше не превращает `409` в `500`
- [Берлин, 29 марта: слоты записи при переводе часов без прибавления 24 часов](notes/slotbook-dst-slots.ru.md): даты по полуночам UTC и 23 вектора, общие для Go и TypeScript
- [Поздний ack оказался мелочью: fencing token из срока аренды в очереди на Redis](notes/relay-lease-fencing.ru.md): вторая часть, где fencing token вскрывает ошибку посерьёзнее
- [Очередь на Redis атомарно забирала задачи и всё равно отдавала одну и ту же двум воркерам](notes/relay-queue-races.ru.md): две гонки, тест, который ловит их при каждом запуске, и исправление
- [Очередь доставок Relay на двух sorted set в Redis с арендой задачи на 120 секунд](notes/relay-redis-lease-queue.ru.md)
- [Ротация refresh-токенов в FlowDesk через условный UPDATE и блокировку между вкладками](notes/flowdesk-refresh-rotation.ru.md)
- [Как Zernolist считает цену заказа на сервере и принимает каждый платёж в Telegram Stars один раз](notes/zernolist-stars-payments.ru.md)
- [Как DocMind отдаёт источники раньше первого токена и превращает [n] в ссылки на лету](notes/docmind-sources-before-tokens.ru.md)
- [Как Pulse считает посетителей через суточную соль и HyperLogLog](notes/pulse-hyperloglog.ru.md)

Написать: Telegram [@sinnercode](https://t.me/sinnercode), [портфолио](https://sinnercode228.github.io/portfolio/).
