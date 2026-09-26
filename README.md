# Грешный Котик

[English version](#english)

Full-stack разработчик: Python и TypeScript. Больше 7 лет пишу код на заказ, в основном на фрилансе, и сейчас беру новые проекты.
В проектах ниже много кода на случай, если вебхук или платёж доставлен дважды, два refresh-запроса пришли одновременно, воркер упал посреди задачи или оборвалось соединение.

На заказ делаю:
- связки сайта, CRM и мессенджеров: amoCRM, Bitrix24, Tilda, МойСклад, Google Sheets, Telegram;
- Telegram-ботов и Mini App, в том числе с оплатой в Stars;
- веб-приложения и админки: CRM, личные кабинеты, дашборды;
- парсеры, выгрузки и отчёты в Excel, где итоги считают формулы;
- ответы по базе знаний со ссылками на источник и автоматизацию в n8n.

Коммерческий код остаётся у заказчиков, поэтому здесь открытые демо на выдуманных данных, собранные вокруг тех же задач. Сроки и цены — на [странице портфолио](https://sinnercode228.github.io/portfolio/#services).

<p>
  <a href="https://sinnercode228.github.io/pulse-analytics/"><img src="https://raw.githubusercontent.com/sinnercode228/portfolio/main/assets/projects/pulse-analytics/cover.webp" width="32%" alt="Pulse, дашборд аналитики"></a>
  <a href="https://sinnercode228.github.io/tg-shop-miniapp/"><img src="https://raw.githubusercontent.com/sinnercode228/portfolio/main/assets/projects/tg-shop-miniapp/cover.webp" width="32%" alt="Zernolist, магазин в Telegram"></a>
  <a href="https://sinnercode228.github.io/flowdesk-crm/"><img src="https://raw.githubusercontent.com/sinnercode228/portfolio/main/assets/projects/flowdesk-crm/cover.webp" width="32%" alt="FlowDesk, канбан сделок"></a>
</p>

- **[Pulse](https://github.com/sinnercode228/pulse-analytics)** · [демо](https://sinnercode228.github.io/pulse-analytics/)\
  Аналитика сайтов без cookies с трекером на 843 байта (560 в gzip) и аптайм-мониторингом. Посетителей в каждой строке роллапа считает [HyperLogLog-скетч](https://github.com/sinnercode228/pulse-analytics/blob/main/packages/core/src/hll.ts), который до 512 хэшей хранит их точным множеством, а дальше переходит на 2048 регистров со стандартной ошибкой около 2,3%.\
  TypeScript, Fastify, node:sqlite, WebSocket, React, uPlot

- **[Relay](https://github.com/sinnercode228/integration-hub)** · [демо](https://sinnercode228.github.io/integration-hub/)\
  Принимает вебхуки Tilda, amoCRM и Bitrix24 и разносит их в Telegram, Google Sheets, amoCRM и на почту. Очередь своя, [на sorted sets в Redis](https://github.com/sinnercode228/integration-hub/blob/main/backend/src/relay/queue/redis.py): взятая задача переезжает в отдельный ZSET с дедлайном через 120 секунд, и если воркер упадёт, после дедлайна она вернётся в очередь.\
  Python, FastAPI, Redis, SQLite, React

- **[DocMind](https://github.com/sinnercode228/docmind-rag)** · [демо](https://sinnercode228.github.io/docmind-rag/)\
  RAG-ассистент по PDF, DOCX и веб-страницам. Источники приходят по SSE [раньше первого токена ответа](https://github.com/sinnercode228/docmind-rag/blob/main/backend/src/docmind/rag/service.py#L110-L111), и когда в тексте появляется сноска `[n]`, её карточка уже на экране.\
  Python, FastAPI, pgvector, Claude или OpenAI-совместимый API, React, бот на aiogram

- **[Zernolist](https://github.com/sinnercode228/tg-shop-miniapp)** · [демо](https://sinnercode228.github.io/tg-shop-miniapp/)\
  Магазин кофе и чая внутри Telegram с оплатой в Stars. Сервер [пересчитывает каждый заказ по каталогу](https://github.com/sinnercode228/tg-shop-miniapp/blob/main/bot/tgshop/services/orders.py#L76-L97), поэтому тест шлёт `price: 1` и всё равно получает подытог 1290 ₽.\
  Mini App на React и Tailwind, бот и API на aiogram 3 + FastAPI, SQLAlchemy

- **[FlowDesk](https://github.com/sinnercode228/flowdesk-crm)** · [демо](https://sinnercode228.github.io/flowdesk-crm/)\
  CRM с канбаном сделок. Refresh-токены ротируются, повторно использованный токен отзывает всю сессию, а гонку двух одновременных refresh закрывает [условный `updateMany` в транзакции](https://github.com/sinnercode228/flowdesk-crm/blob/main/server/src/modules/auth/auth.service.ts#L44-L51).\
  Next.js, Fastify, Prisma, PostgreSQL, TanStack Query, dnd-kit

Демо лежат на GitHub Pages статикой, бэкенд в них заменён кодом в браузере; у Relay события и сбои генерирует отдельная симуляция. У DocMind в демо нет LLM: поиск идёт по BM25, ответ собирается из найденных предложений.

[n8n-home-ai-workflows](https://github.com/sinnercode228/n8n-home-ai-workflows) — четыре воркфлоу для n8n: заметки со встреч в Notion через Claude, дайджест семейных календарей, вопросы по домашним документам на локальных Ollama и Qdrant, обработчик ошибок. С живыми аккаунтами Anthropic, Notion и Google их не запускал; мок-прогоны исполняют настоящий `jsCode` из JSON воркфлоу в `vm`.

В [portfolio](https://github.com/sinnercode228/portfolio) ещё четыре демо поменьше: лендинг с калькулятором стоимости дома ([демо](https://sinnercode228.github.io/portfolio/landing-calculator/)), асинхронный парсер books.toscrape.com с выгрузкой в XLSX, CSV и JSON, Telegram-бот для заявок и Excel-дашборд из CSV, где каждая цифра считается формулой.

### Стек

Python: FastAPI, aiogram 3, SQLAlchemy 2, Pydantic, httpx, openpyxl\
TypeScript: Fastify, Prisma, zod, React 19, Next.js, Vite, TanStack Query, Zustand, Tailwind\
Данные: PostgreSQL, pgvector, SQLite, Redis\
LLM: Claude API, OpenAI-совместимые API, Ollama, Qdrant, n8n\
Проверки и инфраструктура: pytest, Vitest, mypy (strict), ruff, GitHub Actions, Docker Compose, nginx

Тестов во всех демо 1047, в каждом репозитории их гоняет GitHub Actions. У каждого проекта рядом с README лежит английская версия.

Обсудить задачу: Telegram [@sinnercode](https://t.me/sinnercode)

---

### English

Full-stack developer, Python and TypeScript. I've been doing client work for more than 7 years, mostly freelance, and I'm taking on new projects.
The projects above spend a lot of code on failure cases: a webhook or payment delivered twice, two refresh requests at once, a worker dying mid-job, a dropped connection.

What I build for clients: integrations between sites, CRMs and messengers (amoCRM, Bitrix24, Tilda, MoySklad, Google Sheets, Telegram), Telegram bots and Mini Apps with Stars payments, web apps and admin panels, scrapers and Excel reports where the totals are formulas, answers over a knowledge base with source citations, and n8n automation.

Client code stays with clients, so these are open demos on made-up data, built around the same kinds of tasks. Every project has an English README next to the Russian one.

- [Pulse](https://github.com/sinnercode228/pulse-analytics) · [demo](https://sinnercode228.github.io/pulse-analytics/): cookieless web analytics with an 843-byte tracker (560 bytes gzipped) and uptime monitoring. Each rollup row carries a HyperLogLog sketch: an exact set up to 512 hashes, then 2048 registers with about 2.3% standard error.
- [Relay](https://github.com/sinnercode228/integration-hub) · [demo](https://sinnercode228.github.io/integration-hub/): takes webhooks from Tilda, amoCRM and Bitrix24 and delivers them to Telegram, Google Sheets, amoCRM and e-mail. The queue runs on Redis sorted sets; a job that a worker takes gets a 120-second lease and goes back to the queue if the worker dies.
- [DocMind](https://github.com/sinnercode228/docmind-rag) · [demo](https://sinnercode228.github.io/docmind-rag/): RAG over PDF, DOCX and web pages. Sources arrive over SSE before the first answer token, so a `[n]` citation already has its card on screen when it appears.
- [Zernolist](https://github.com/sinnercode228/tg-shop-miniapp) · [demo](https://sinnercode228.github.io/tg-shop-miniapp/): a coffee and tea shop inside Telegram with Stars payments. The server re-prices every order from the catalog, so a test that sends `price: 1` still gets a 1290 ₽ subtotal.
- [FlowDesk](https://github.com/sinnercode228/flowdesk-crm) · [demo](https://sinnercode228.github.io/flowdesk-crm/): a CRM with a deals kanban. Refresh tokens rotate, a reused token revokes the whole session, and a conditional `updateMany` inside a transaction closes the race between two concurrent refreshes.

The demos are static builds on GitHub Pages with the backend replaced by code in the browser. Relay's events and failures come from a separate simulation, and DocMind's demo has no LLM: it searches with BM25 and assembles answers from the sentences it finds.

Also: [n8n workflows](https://github.com/sinnercode228/n8n-home-ai-workflows) (meeting notes to Notion with Claude, document Q&A on local Ollama and Qdrant; mock-run, not tried against live accounts) and four smaller demos in [portfolio](https://github.com/sinnercode228/portfolio): a landing page with a house cost calculator, an async scraper, a lead-capture bot and an Excel dashboard built on formulas. 1,047 tests across all of them, run by GitHub Actions.

Services, timelines and prices: [sinnercode228.github.io/portfolio](https://sinnercode228.github.io/portfolio/?lang=en#services). Contact: Telegram [@sinnercode](https://t.me/sinnercode).
