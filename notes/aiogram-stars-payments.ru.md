# Звёзды списаны, а on_paid не вызван: где теряются платежи Telegram Stars в aiogram

[English (dev.to)](https://dev.to/sinnercode228/the-stars-were-charged-and-onpaid-never-ran-where-telegram-stars-payments-get-lost-in-aiogram-5m2) · **Русский**

Покупатель платит 100 звёзд, Telegram их списывает, а бот отвечает на сообщение об оплате: «Sorry, I don't know this command». Обработчик `on_paid`, который должен выдать товар, не вызван, в базе о платеже ни строки. Так вела себя моя библиотека aiogram-stars за день до релиза 0.1.0 в любом боте, где роутер с обработчиком «на всё» подключён раньше неё.

В релиз эта ошибка не попала. Перед тем как выложить первую версию, я ещё раз прошёл по коду с одним вопросом: где здесь может потеряться платёж? Нашлось девять ошибок. Каждая исправлена отдельным коммитом, и в каждом коммите есть тест, который на коде до исправления падает. Ниже подробно четыре истории: `asyncio.wait_for`, который не держал десятисекундный дедлайн pre-checkout, роутер «на всё», остановка `dp.run_polling()` посреди `on_paid` и flood control в очереди возвратов. Остальные пять — коротким списком, а в конце то, что не решено до сих пор.

Код лежит в репозитории [sinnercode228/aiogram-stars](https://github.com/sinnercode228/aiogram-stars). Ссылки на строки библиотеки ведут на коммит [`d601bc3`](https://github.com/sinnercode228/aiogram-stars/tree/d601bc348503130979eee6c2313de45265dbb23d), из которого собран релиз 0.1.0, кроме двух, которые показывают код до исправления. Библиотеку я вынес из платёжного кода [Zernolist](https://github.com/sinnercode228/tg-shop-miniapp), демо-магазина кофе в Telegram Mini App. Все прогоны в тексте мои: aiogram 3.31.0 и Python 3.13, а скрипт с `wait_for` я запускал ещё и на 3.10–3.12.

> **Коротко.** Четыре ошибки, о которых ниже, по строке на каждую:
>
> 1. `asyncio.wait_for` не держал десятисекундный дедлайн pre-checkout: по таймауту он ждёт, пока отменённая `check` закончится, и в моём прогоне ответ ушёл через 11,01 с после начала обработки.
> 2. Роутер «на всё», подключённый до `stars.setup(dp)`, забирал сообщение с `successful_payment`: `on_paid` не вызван, в хранилище пусто.
> 3. Остановка `dp.run_polling()` обрывала `on_paid`: перед выходом `asyncio.run` отменяет задачи, которые ещё живы, строка оставалась `processing`, а Telegram этот апдейт больше не присылает.
> 4. Flood control в очереди возвратов: 429 учитывался только для того возврата, который на него наткнулся, проход шёл дальше, и каждый возврат терял одну из восьми попыток на вызов без шансов на успех.

## Как устроена оплата звёздами

Оплата приходит двумя апдейтами. Первый — `pre_checkout_query`: Telegram спрашивает бота, проводить ли платёж, и ждёт ответа `answerPreCheckoutQuery` 10 секунд. Не ответил вовремя — оплата у покупателя не проходит. Второй — сообщение с `successful_payment`: звёзды уже списаны, и бот должен выдать товар. Вернуть их можно только вызовом `refundStarPayment`.

В aiogram-stars с вашей стороны две функции. `check` решает, принимать ли платёж, `on_paid` выдаёт товар. Так выглядит середина быстрого старта из [README](https://github.com/sinnercode228/aiogram-stars/blob/d601bc348503130979eee6c2313de45265dbb23d/README.ru.md#L72-L88):

```python
async def check(checkout: Checkout) -> None:
    if stock["coffee"] == 0:
        raise RejectPayment("Sold out, sorry.")  # declines the pre-checkout query


async def on_paid(payment: Payment, message: Message) -> None:
    if stock["coffee"] == 0:  # the last bag went while this buyer was paying
        raise RejectPayment("Sold out. Your Stars are on their way back.")  # queues a refund
    stock["coffee"] -= 1
    await message.answer(f"Order {payment.order_id} is paid, thank you!")


stars = StarsPayments(
    MemoryStorage(), secret=os.environ["STARS_SECRET"], check=check, on_paid=on_paid
)
dp = Dispatcher()
stars.setup(dp)
```

Остальное делает библиотека. Payload инвойса подписан HMAC, в подпись входят номер заказа и сумма, и чужой или изменённый payload отклоняется ещё до `check`. На pre-checkout библиотека отвечает раньше дедлайна. `on_paid` вызывается один раз на каждый `telegram_payment_charge_id`: перед вызовом в таблицу пишется строка со статусом `processing` и арендой (`INSERT ... ON CONFLICT (charge_id) DO NOTHING`), и дальше идёт только тот обработчик, который её вставил. Возврат сначала записывается в таблицу, а отправляет его фоновая задача с повторами.

Так задумано. Дальше о том, где это не выполнялось.

![Два апдейта платежа звёздами: pre_checkout_query с десятью секундами на ответ, потом successful_payment, дерево роутеров и on_paid, и очередь возвратов, которая отправляет refundStarPayment. Красные метки от 1 до 4 показывают, где сработала каждая из четырёх ошибок.](https://raw.githubusercontent.com/sinnercode228/aiogram-stars/main/docs/images/flow.ru.png)

## 1. На ответ десять секунд, а ответ уходил через одиннадцать

`check` обычно ходит в базу: есть ли товар, не оплачен ли уже заказ. Чтобы медленная база не съела все десять секунд, библиотека даёт `check` время `checkout_timeout`, по умолчанию 8 секунд, а оставшиеся две оставляет на сам запрос `answerPreCheckoutQuery`. Первая версия делала это через `asyncio.wait_for` ([`_payments.py#L326-L344` на `4bc1436`](https://github.com/sinnercode228/aiogram-stars/blob/4bc143683483226920382e886409c4ebb60cd30d/src/aiogram_stars/_payments.py#L326-L344)):

```python
        try:
            await asyncio.wait_for(self._check(checkout, data), self._checkout_timeout)
        except RejectPayment as exc:
            log.info("check declined order %s: %s", order_id, exc.reason)
            return False, exc.reason
        except asyncio.TimeoutError:
            log.error(
                "check for order %s took longer than %s s; declined pre-checkout query %s",
                order_id,
                self._checkout_timeout,
                query.id,
            )
            return False, self._checkout_error
        # ... (other exceptions cut)
        return True, None
```

**Почему ломается.** Выглядит правильно: не уложился — отказ с `checkout_error`. Но по таймауту `wait_for` сначала отменяет задачу, ждёт, пока она закончится, и только потом бросает `TimeoutError`. Отмена в asyncio — это просьба. В точке, где `check` ждёт, поднимается `CancelledError`, а дальше выполняются её `finally` и `except` со всеми их `await`. Если `check` держит соединение с базой и в `finally` откатывает транзакцию на том же зависшем соединении, `wait_for` ждёт и этот откат.

**Воспроизведение.** Это видно и без aiogram:

```python
import asyncio
import time


async def slow_to_stop() -> None:
    try:
        await asyncio.sleep(30)  # a stock query that hangs
    finally:
        await asyncio.sleep(0.5)  # a rollback on the same hung connection


async def ignores_cancel() -> None:
    try:
        await asyncio.sleep(30)
    except asyncio.CancelledError:
        await asyncio.sleep(0.1)  # and returns as if all was well


async def main() -> None:
    for check in (slow_to_stop, ignores_cancel):
        started = time.monotonic()
        try:
            await asyncio.wait_for(check(), timeout=0.05)
            outcome = "returned: the payment would be approved"
        except asyncio.TimeoutError:
            outcome = "TimeoutError"
        print(f"{check.__name__}: {outcome} after {time.monotonic() - started:.2f} s")


asyncio.run(main())
```

На Python 3.10, 3.11, 3.12 и 3.13 вывод одинаковый:

```text
slow_to_stop: TimeoutError after 0.55 s
ignores_cancel: returned: the payment would be approved after 0.15 s
```

Таймаут 0,05 с, а `TimeoutError` приходит через 0,55 с. Вторая строка хуже: если `check` проглотила отмену и вернулась как обычно, `wait_for` возвращает её результат, и `TimeoutError` нет вовсе. Это прямо написано в docstring `wait_for` в [Python 3.13](https://github.com/python/cpython/blob/v3.13.12/Lib/asyncio/tasks.py#L470-L471): «If the task suppresses the cancellation and returns a value instead, that value is returned». Проглотить отмену легко: `CancelledError` наследует `BaseException`, и его ловят голый `except:` и `except BaseException`. В старом коде после этого шло `return True, None`, то есть платёж одобрялся, хотя `check` прервали посреди работы.

С настройками по умолчанию я прогнал старый код на поддельной сессии Bot API: `check` висит, а откат в `finally` занимает 3 секунды. Лог пишет «took longer than 8.0 s», но ответ ушёл через 11,01 с после начала обработки. Это больше десяти секунд, которые Telegram даёт на ответ, и это ещё без времени, за которое апдейт дошёл до бота.

![Шкала времени ответа на pre-checkout с настройками по умолчанию. До исправления, с asyncio.wait_for: check висит, по таймауту на 8 с в check поднимается CancelledError, wait_for ждёт трёхсекундный откат в её finally, и ответ уходит через 11,01 с, позже десяти секунд, которые даёт Telegram. После исправления, с asyncio.wait: ответ уходит через 8,01 с, а отменённую check обработчик дожидается уже после ответа.](https://raw.githubusercontent.com/sinnercode228/aiogram-stars/main/docs/images/waitfor-timeline.ru.png)

**Исправление.** Код после исправления ([`_payments.py#L391-L409`](https://github.com/sinnercode228/aiogram-stars/blob/d601bc348503130979eee6c2313de45265dbb23d/src/aiogram_stars/_payments.py#L391-L409)) запускает `check` отдельной задачей и ждёт её через `asyncio.wait`. По таймауту `wait` ничего не отменяет и ни на что не ждёт, а возвращает управление:

```python
        # A task rather than wait_for, which waits for the cancelled check to stop: a check that
        # is slow to clean up, or that ignores its cancellation, must not delay the answer.
        task = asyncio.ensure_future(check(checkout, data))
        try:
            await asyncio.wait({task}, timeout=self._checkout_timeout)
        except asyncio.CancelledError:
            task.cancel()
            raise
        if not task.done():
            task.cancel()
            log.error(
                "check for order %s took longer than %s s; declined pre-checkout query %s",
                order_id,
                self._checkout_timeout,
                query_id,
            )
            return _Verdict(
                ok=False, error=self._checkout_error, order_id=order_id, late_check=task
            )
```

Отменённая задача попадает в вердикт как `late_check`, и ждут её только после того, как ответ отправлен ([`#L363-L370`](https://github.com/sinnercode228/aiogram-stars/blob/d601bc348503130979eee6c2313de45265dbb23d/src/aiogram_stars/_payments.py#L363-L370)):

```python
        verdict = await self._decide(query, data)
        try:
            await bot.answer_pre_checkout_query(
                pre_checkout_query_id=query.id, ok=verdict.ok, error_message=verdict.error
            )
        finally:
            if verdict.late_check is not None:
                await _outlast(verdict.late_check, verdict.order_id)
```

[`_outlast`](https://github.com/sinnercode228/aiogram-stars/blob/d601bc348503130979eee6c2313de45265dbb23d/src/aiogram_stars/_payments.py#L65-L73) дожидается задачи и пишет предупреждение, если она так и не остановилась. Брошенной задача не остаётся: обработчик ждёт её до конца, и ошибка, с которой она закончилась, попадает в лог. Это проверяет отдельный тест. Отмена самого обработчика отменяет и `check`, для этого ветка `except asyncio.CancelledError`.

У перехода на задачу есть и второе следствие: вердикт теперь читается из неё явно. Задача отменилась сама — отказ. `RejectPayment` — отказ с причиной, которую увидит покупатель. Любое другое исключение — отказ с `checkout_error`. Вернула что угодно кроме `None` — тоже отказ. До коммита `668bf9d` `check`, которая возвращала `False`, одобряла платёж: результат никто не смотрел.

**Тест.** [`tests/test_pre_checkout.py#L165-L191`](https://github.com/sinnercode228/aiogram-stars/blob/d601bc348503130979eee6c2313de45265dbb23d/tests/test_pre_checkout.py#L165-L191):

```python
async def test_check_that_is_slow_to_stop_is_answered_at_the_deadline(
    bot: Bot, session: FakeSession, caplog: pytest.LogCaptureFixture
) -> None:
    cleaned_up = asyncio.Event()

    async def check(checkout: Checkout) -> None:
        try:
            await asyncio.sleep(30)
        finally:
            # As slow as the work itself, like a rollback on a database connection that hangs.
            await asyncio.sleep(0.5)
            cleaned_up.set()

    dp = dispatcher(make_stars(check=check, checkout_timeout=0.05, checkout_error="Too slow"))

    started = time.monotonic()
    with caplog.at_level(logging.WARNING, logger="aiogram_stars"):
        handling = asyncio.create_task(
            dp.feed_update(bot, pre_checkout_update(bot, SIGNER.sign("order-1", 10), 10))
        )
        assert await answered(session) - started < 0.4
        assert answer(session) == (False, "Too slow")
        assert not cleaned_up.is_set()
        await handling

    assert cleaned_up.is_set(), "the handler lets the cancelled check finish after answering"
    assert "check for order order-1 took longer than 0.05 s" in caplog.text
```

На коде до исправления (`4bc1436`) он падает на первой проверке: ответ ушёл через 0,56 с при таймауте 0,05 с.

```text
E   assert (473.466552522 - 472.908190609) < 0.4
```

[Соседний тест](https://github.com/sinnercode228/aiogram-stars/blob/d601bc348503130979eee6c2313de45265dbb23d/tests/test_pre_checkout.py#L194-L209), где `check` глотает отмену, на старом коде падает с `AssertionError: assert (True, None) == (False, 'Too slow')`: платёж одобрен. А в прогоне с настройками по умолчанию исправленный код отвечает через 8,01 с.

## 2. Роутер «на всё» забирал оплату себе

**Почему ломается.** aiogram ищет обработчик по дереву роутеров: сначала обработчики самого роутера, потом вложенные роутеры по порядку, и первый подошедший забирает апдейт ([`router.py#L199-L202`](https://github.com/aiogram/aiogram/blob/v3.31.0/aiogram/dispatcher/router.py#L199-L202) в aiogram 3.31.0):

```python
        for router in self.sub_routers:
            response = await router.propagate_event(update_type=update_type, event=event, **kwargs)
            if response is not UNHANDLED:
                break
```

`include_router` присваивает роутеру `parent_router` ([`#L278`](https://github.com/aiogram/aiogram/blob/v3.31.0/aiogram/dispatcher/router.py#L278)), и этот сеттер добавляет его в конец списка ([`#L252-L253`](https://github.com/aiogram/aiogram/blob/v3.31.0/aiogram/dispatcher/router.py#L252-L253)). А `stars.setup(dp)` в первой версии делал ровно `router.include_router(self._router)`. Частый приём в ботах — роутер с ответом на непонятные команды, `@router.message()` без фильтров. Сообщение с `successful_payment` — тоже `message`, и если такой роутер подключён до `setup`, оплату забирает он.

**Воспроизведение.** Я собрал такой бот на коде до исправления (`668bf9d`): роутер `fallback` с одним обработчиком, который отвечает «Sorry, I don't know this command», потом `dp.include_router(fallback)`, `stars.setup(dp)`, pre-checkout и оплата. Результат:

```text
sub_routers: ['fallback', 'aiogram-stars']
pre-checkout ok=True; on_paid calls: 0; stored: None; bot replied: ["Sorry, I don't know this command"]
```

Pre-checkout прошёл: обработчика `pre_checkout_query` у `fallback` нет, и ответила библиотека. Telegram списал звёзды и прислал `successful_payment`, а его забрал `fallback`. `on_paid` не вызван, в хранилище пусто, покупатель получил «не знаю такой команды». Если в `fallback` есть ещё и `@router.pre_checkout_query()` без фильтров, как в тесте ниже, на pre-checkout не отвечает никто, и через 10 секунд Telegram отменяет оплату.

**Исправление.** Две строки в `setup` после `include_router` ([`_payments.py#L202-L204`](https://github.com/sinnercode228/aiogram-stars/blob/d601bc348503130979eee6c2313de45265dbb23d/src/aiogram_stars/_payments.py#L202-L204)):

```python
        router.include_router(self._router)
        router.sub_routers.remove(self._router)
        router.sub_routers.insert(0, self._router)
```

`include_router` по-прежнему не даёт подключить роутер второй раз (aiogram бросает `RuntimeError`), а потом роутер библиотеки переезжает в начало списка. Остальные роутеры сохраняют свой порядок. Обработчики, зарегистрированные на самом `dp`, всё равно срабатывают раньше ([`#L183-L197`](https://github.com/aiogram/aiogram/blob/v3.31.0/aiogram/dispatcher/router.py#L183-L197)), и это оставлено намеренно: так бот может сам принять оплату по инвойсам старого формата, выставленным до перехода на библиотеку. [Отдельный тест](https://github.com/sinnercode228/aiogram-stars/blob/d601bc348503130979eee6c2313de45265dbb23d/tests/test_lifecycle.py#L267-L282) проверяет, что `@dp.message(F.successful_payment.invoice_payload.startswith("legacy:"))` получает такой платёж, а библиотека его не записывает. Обратная сторона осталась: обработчик «на всё», повешенный прямо на `dp`, по-прежнему заберёт оплату раньше библиотеки.

**Тест.** [`tests/test_lifecycle.py#L235-L264`](https://github.com/sinnercode228/aiogram-stars/blob/d601bc348503130979eee6c2313de45265dbb23d/tests/test_lifecycle.py#L235-L264):

```python
async def test_catch_all_handlers_in_routers_included_earlier_do_not_swallow_payments(
    bot: Bot, session: FakeSession
) -> None:
    paid: list[Payment] = []

    async def on_paid(payment: Payment) -> None:
        paid.append(payment)

    stars = make_stars(on_paid=on_paid)
    fallback = Router(name="fallback")

    @fallback.message()
    async def unknown_command(message: Message) -> None:
        return None

    @fallback.pre_checkout_query()
    async def any_query(query: PreCheckoutQuery) -> None:
        return None

    dp = Dispatcher()
    dp.include_router(fallback)
    stars.setup(dp)

    await dp.feed_update(bot, pre_checkout_update(bot, PAYLOAD, 100))
    await dp.feed_update(bot, payment_update(bot, PAYLOAD, 100))

    [answer] = session.calls(AnswerPreCheckoutQuery)
    assert answer.ok is True
    assert [payment.charge_id for payment in paid] == ["charge-1"]
    assert dp.sub_routers[-1] is fallback, "the other routers keep their order"
```

На `668bf9d` он падает на распаковке: ответа на pre-checkout нет вовсе.

```text
E   ValueError: not enough values to unpack (expected 1, got 0)
```

## 3. Остановка бота обрывала on_paid, а Telegram этот апдейт больше не присылал

`on_paid` выдаёт товар: пишет в базу, отправляет сообщение, бывает, ходит во внешний сервис. Если в это время бот перезапускают (деплой, `systemctl restart`, `docker stop`), старый код оставлял платёж в `processing` навсегда.

**Почему ломается.** Объясняют это три места в aiogram 3.31.0.

Первое: при polling каждый апдейт по умолчанию (`handle_as_tasks=True`) обрабатывается в отдельной задаче, и цикл чтения её не ждёт ([`dispatcher.py#L398-L410`](https://github.com/aiogram/aiogram/blob/v3.31.0/aiogram/dispatcher/dispatcher.py#L398-L410)):

```python
                handle_update = self._process_update(bot=bot, update=update, **kwargs)
                if handle_as_tasks:
                    # ... (the tasks_concurrency_limit branch cut)
                    else:
                        handle_update_task = asyncio.create_task(handle_update)

                    self._handle_update_tasks.add(handle_update_task)
                    handle_update_task.add_done_callback(self._handle_update_tasks.discard)
```

Задачи складываются в `_handle_update_tasks`, но больше это множество нигде не используется, в том числе при остановке.

Второе: апдейт подтверждается следующим `getUpdates` со сдвинутым `offset` ([`#L248-L254`](https://github.com/aiogram/aiogram/blob/v3.31.0/aiogram/dispatcher/dispatcher.py#L248-L254)):

```python
            for update in updates:
                yield update
                # The getUpdates method returns the earliest 100 unconfirmed updates.
                # To confirm an update, use the offset parameter when calling getUpdates
                # All updates with update_id less than or equal to offset will be marked
                # as confirmed on the server and will no longer be returned.
                get_updates.offset = update.update_id + 1
```

Раз обработчик ушёл в задачу, следующий `getUpdates` отправляется сразу, пока `on_paid` ещё работает. В моём прогоне ниже к началу `on_paid` для апдейта 500 бот уже отправил `getUpdates` с `offset=501`. Для Telegram апдейт обработан, второй раз он его не пришлёт.

Третье: `run_polling` — это `asyncio.run(self.start_polling(...))` ([`#L677`](https://github.com/aiogram/aiogram/blob/v3.31.0/aiogram/dispatcher/dispatcher.py#L677)), а с установленным uvloop на Python 3.11+ — `asyncio.Runner`, который на выходе делает то же самое. На остановке `start_polling` отменяет задачи чтения апдейтов, вызывает shutdown-хуки и закрывает сессию бота ([`#L614-L631`](https://github.com/aiogram/aiogram/blob/v3.31.0/aiogram/dispatcher/dispatcher.py#L614-L631)), а задачи обработчиков не ждёт. Потом `asyncio.run` перед выходом отменяет все задачи, которые ещё живы ([`runners.py#L198-L206`](https://github.com/python/cpython/blob/v3.13.12/Lib/asyncio/runners.py#L198-L206)), и `on_paid` получает `CancelledError`. `CancelledError` — не `Exception`, ветка `except Exception` в `_handle_payment` его не ловит, результат платежа не записывается, строка остаётся `processing`. Если `on_paid` собирался бросить `RejectPayment`, потому что товар кончился, в очередь не попадает и возврат.

**Воспроизведение.** Прогон: `dp.run_polling(bot)` на поддельной сессии, хранилище — `SqlAlchemyStorage` на файле SQLite, в очереди один апдейт с оплатой. `on_paid` спит секунду, это «выдача товара», а как только он начался, процесс шлёт себе SIGTERM. Код до исправления (`607a2c7`):

```text
 0.04s on_paid started; getUpdates sent with offsets [None, 501]
 0.04s SIGTERM
Received SIGTERM signal
 0.04s on_paid cancelled
 0.04s run_polling returned
[('charge-1', 'processing')]
```

Что обрывает именно `asyncio.run`, а не aiogram, я проверил отдельно: если вызвать `await dp.start_polling(bot)` в своей корутине и после него подождать две секунды, `on_paid` доходит до конца через 1,04 с после старта, и платёж записывается как `accepted`.

Аренда (`payment_lease`) здесь не помогает. Она позволяет другому обработчику забрать платёж, если апдейт с тем же `charge_id` придёт снова, а он не придёт.

![Два прогона dp.run_polling, где SIGTERM приходит на 0,04 с, как только начался on_paid и уже после того, как ушёл getUpdates с offset=501. До исправления start_polling останавливается, не дожидаясь обработчиков, asyncio.run отменяет on_paid, и строка остаётся processing. После исправления stop() в shutdown-хуке ждёт on_paid, тот завершается на 1,04 с, и платёж записывается как accepted.](https://raw.githubusercontent.com/sinnercode228/aiogram-stars/main/docs/images/shutdown-timeline.ru.png)

**Исправление.** Оно состоит из двух частей. Обработчик оплаты запоминает свою задачу ([`_payments.py#L446-L455`](https://github.com/sinnercode228/aiogram-stars/blob/d601bc348503130979eee6c2313de45265dbb23d/src/aiogram_stars/_payments.py#L446-L455)):

```python
        handler = asyncio.current_task()
        if handler is None:  # pragma: no cover - aiogram runs handlers in tasks
            await self._handle_payment(message, paid, data)
            return
        key = object()  # one per call: a handler may feed another update from its own task
        self._handling[key] = (handler, paid.telegram_payment_charge_id)
        try:
            await self._handle_payment(message, paid, data)
        finally:
            del self._handling[key]
```

А `stop()`, который вызывает shutdown-хук, сначала ждёт эти задачи, но не дольше `payment_lease` ([`#L216-L240`](https://github.com/sinnercode228/aiogram-stars/blob/d601bc348503130979eee6c2313de45265dbb23d/src/aiogram_stars/_payments.py#L216-L240)):

```python
    async def stop(self) -> None:
        # ... (docstring cut)
        await self._wait_for_payments()
        await self._outbox.stop()

    async def _wait_for_payments(self) -> None:
        current = asyncio.current_task()
        handlers = {task: charge for task, charge in self._handling.values() if task is not current}
        if not handlers:
            return
        log.info("waiting for %d payment(s) being handled to finish", len(handlers))
        _, pending = await asyncio.wait(handlers, timeout=self._payment_lease.total_seconds())
        for task in pending:
            log.error(
                "payment %s is still being handled at shutdown; if the process exits now, "
                "on_paid is cut off and the payment stays %s",
                handlers[task],
                PaymentStatus.PROCESSING.value,
            )
```

Shutdown-хуки aiogram вызывает раньше, чем закрывает сессию бота, так что `on_paid` в это время ещё может отправлять сообщения. Проверка `task is not current` нужна для `stop()`, вызванного изнутри `on_paid`: иначе он ждал бы сам себя, и на это тоже есть тест.

Тот же прогон после исправления:

```text
 0.04s on_paid started; getUpdates sent with offsets [None, 501]
 0.04s SIGTERM
Received SIGTERM signal
 1.04s on_paid finished
 1.05s run_polling returned
[('charge-1', 'accepted')]
```

**Тест.** [`tests/test_lifecycle.py#L291-L311`](https://github.com/sinnercode228/aiogram-stars/blob/d601bc348503130979eee6c2313de45265dbb23d/tests/test_lifecycle.py#L291-L311) запускает `start_polling` с одним апдейтом в очереди, ждёт начала `on_paid`, останавливает polling и смотрит на статус:

```python
async def test_shutdown_waits_for_a_payment_being_handled(bot: Bot, session: FakeSession) -> None:
    # dp.run_polling() ends in asyncio.run(), which cancels the handlers still running, and
    # Telegram does not send the update again: on_paid would be cut off for good.
    started = asyncio.Event()

    async def on_paid(payment: Payment) -> None:
        started.set()
        await asyncio.sleep(0.2)

    stars = make_stars(on_paid=on_paid, refunds=QUIET)
    dp = Dispatcher()
    stars.setup(dp)
    polling = await start_polling_with_a_payment(dp, bot, session)
    await asyncio.wait_for(started.wait(), 2)

    await dp.stop_polling()
    await polling

    payment = await stars.get_payment("charge-1")
    assert payment is not None
    assert payment.status is PaymentStatus.ACCEPTED
```

На `607a2c7`:

```text
E   AssertionError: assert <PaymentStatus.PROCESSING: 'processing'> is <PaymentStatus.ACCEPTED: 'accepted'>
```

Ожидание ограничено `payment_lease`, по умолчанию это 10 минут. Супервизор, который столько не ждёт и добивает процесс через SIGKILL, возвращает ту же проблему; об этом в разделе «Что осталось открытым».

## 4. Flood control сжигал попытки всех возвратов разом

Возврат в aiogram-stars — строка в таблице `stars_refunds`. Фоновая задача проходит по возвратам, которым пора: сразу, как только в этом же процессе появился новый возврат, а в остальное время раз в `poll_interval` (10 секунд). Возвраты она берёт по одному: захватывает строку арендой, засчитывает попытку и вызывает `refundStarPayment`. По умолчанию попыток 8, пауза после первой неудачи 30 секунд и дальше удваивается до часа, так что последняя попытка случается примерно через 64 минуты после первой.

**Почему ломается.** На 429 Telegram присылает `retry_after`, aiogram превращает такой ответ в `TelegramRetryAfter`. Первая версия ([`_outbox.py` на `c7a1c60`](https://github.com/sinnercode228/aiogram-stars/blob/c7a1c604bb14c5e89608f78dbf4503833a431301/src/aiogram_stars/_outbox.py#L136-L164)) учитывала его только для того возврата, который на него наткнулся, и в том же проходе шла к следующему:

```python
        except TelegramRetryAfter as exc:
            error = describe(exc)
            wait = timedelta(seconds=exc.retry_after)
        # ... (cut)
        else:
            status = RefundStatus.PENDING
            next_attempt_at = now + max(self._policy.delay(refund.attempts), wait)
```

Но flood control относится к боту, а не к одному платежу. Пока он действует, следующий `refundStarPayment` в том же проходе только снова на него наткнётся, а попытка засчитана уже при захвате строки.

**Воспроизведение.** Прогон на коде до исправления: пять возвратов в очереди, поддельный Bot API отвечает 429 с `retry_after=120` на каждый вызов, один проход очереди:

```text
refundStarPayment calls: 5, attempts: [1, 1, 1, 1, 1]
```

Пять запросов к Telegram во время flood control, и каждый возврат потерял одну из восьми попыток на вызов без шансов на успех. С исправлением:

```text
refundStarPayment calls: 1, attempts: [1, 0, 0, 0, 0]
```

![Один проход очереди: пять возвратов, и на каждый вызов приходит 429 с retry_after=120. До исправления после каждого 429 проход идёт дальше: пять вызовов refundStarPayment, и каждый возврат тратит одну из 8 попыток. После исправления первый 429 ставит на паузу всю очередь, и проход останавливается перед следующим захватом: один вызов, а остальные четыре возврата сохраняют все попытки.](https://raw.githubusercontent.com/sinnercode228/aiogram-stars/main/docs/images/floodcontrol.ru.png)

**Исправление.** Одно поле `_paused_until` на всю очередь ([`_outbox.py#L60-L61`](https://github.com/sinnercode228/aiogram-stars/blob/d601bc348503130979eee6c2313de45265dbb23d/src/aiogram_stars/_outbox.py#L60-L61), [`#L109-L112`](https://github.com/sinnercode228/aiogram-stars/blob/d601bc348503130979eee6c2313de45265dbb23d/src/aiogram_stars/_outbox.py#L109-L112), [`#L159-L164`](https://github.com/sinnercode228/aiogram-stars/blob/d601bc348503130979eee6c2313de45265dbb23d/src/aiogram_stars/_outbox.py#L159-L164)):

```python
        # Flood control applies to the bot, so a TelegramRetryAfter pauses every refund.
        self._paused_until: datetime | None = None

    # ... (cut)

        while not (background and self._stop_requested()):
            now = self._clock()
            if self._paused_until is not None and now < self._paused_until:
                break

    # ... (cut)

    async def _attempt(self, bot: Bot, refund: Refund, token: str) -> Refund | None:
        error, wait = await self._send(bot, refund)
        now = self._clock()
        if wait:
            self._paused_until = now + wait
            log.warning("flood control: no refunds are sent before %s", self._paused_until)
```

Проверка стоит до захвата строки, поэтому во время паузы возврат не захватывается и попытка не тратится. Сам вызов `refundStarPayment` переехал в `_send`, который возвращает ошибку и время ожидания.

**Тест.** [`tests/test_refunds.py#L182-L211`](https://github.com/sinnercode228/aiogram-stars/blob/d601bc348503130979eee6c2313de45265dbb23d/tests/test_refunds.py#L182-L211):

```python
async def test_flood_control_pauses_the_other_refunds_too(
    bot: Bot, session: FakeSession, storage: Storage, clock: Clock
) -> None:
    # Flood control applies to the bot: sending the next refund at once only hits it again.
    stars = outbox(storage, clock, Desk())
    await stars.refund("a", user_id=1)
    clock.advance(seconds=1)
    await stars.refund("b", user_id=2)
    session.fail(
        RefundStarPayment, "Too Many Requests: retry after 120", status=429, retry_after=120
    )

    [hit] = await stars.send_due_refunds(bot)

    assert (hit.charge_id, hit.status) == ("a", RefundStatus.PENDING)
    assert len(session.calls(RefundStarPayment)) == 1
    waiting = await stars.get_refund("b")
    assert waiting is not None
    assert waiting.attempts == 0, "the other refund was not tried during the flood wait"

    clock.advance(seconds=119)
    assert await stars.send_due_refunds(bot) == []
    assert len(session.calls(RefundStarPayment)) == 1

    clock.advance(seconds=1)
    sent = await stars.send_due_refunds(bot)
    # ... (both refunds are done)
```

Здесь поддельный Bot API отвечает 429 только на первый вызов. Старый код (`c7a1c60`) в том же проходе сразу отправил второй возврат, `send_due_refunds` вернул два возврата вместо одного, и тест падает на распаковке: `ValueError: too many values to unpack (expected 1)`. Тест идёт на хранилище в памяти и на SQLite, а если задан `STARS_TEST_POSTGRES_URL`, то и на PostgreSQL.

Пауза живёт в памяти процесса. Если возвраты из одной базы отправляют несколько процессов, каждый узнаёт о flood control сам, на своём первом 429.

## Ещё пять, коротко

- `4bc1436`. Подпись payload сравнивалась через `hmac.compare_digest` над строками, а он бросает `TypeError: comparing strings with non-ASCII characters is not supported`, если в строке есть не-ASCII символы. Payload с тегом из 22 букв «ё» (длина как у настоящего тега) ронял обработчик: pre-checkout оставался без ответа, а оплата с таким payload не записывалась и не возвращалась. Теперь теги сравниваются как байты.
- `668bf9d`. Результат `check` не проверялся, и `return False` одобрял платёж. Теперь одобряет только `None`, а любое другое значение — отказ и ошибка в логе с подсказкой бросать `RejectPayment`.
- `177a858`. `PaymentStatus` и `RefundStatus` унаследованы от `str, Enum`, и `f"{status}"` давал `accepted` на Python 3.10, но `PaymentStatus.ACCEPTED` на 3.11 и новее. Теперь `__str__` возвращает значение.
- `956a0cc`. Пустой `checkout_error` принимался, а Bot API требует `error_message` при `ok=False` («Required if ok is False» в описании метода). Значит, каждый отказ с этим текстом (неверный payload, медленная или упавшая `check`) нарушал бы это требование. Теперь это `ValueError` при создании `StarsPayments`. Пустую причину в `RejectPayment` и раньше не пропускал конструктор этого исключения.
- `b1668c5`. Extras `[sqlite]` и `[postgres]` ставили драйвер, но не SQLAlchemy, и `aiogram_stars.sqlalchemy` после `pip install "aiogram-stars[postgres]"` не импортировался.

## Что осталось открытым

Главное: платёж может застрять в `processing`. Telegram почти никогда не присылает `successful_payment` второй раз. Polling подтверждает апдейт следующим `getUpdates`, как в третьей истории, а вебхук с `handle_in_background=True` (так по умолчанию в `SimpleRequestHandler`) получает 200 сразу. Поэтому если процесс убили внутри `on_paid` (SIGKILL, нехватка памяти, супервизор, который не ждёт `payment_lease`), платёж так и останется в `processing`. То же будет, если база отказала в момент записи результата. Если же она отказала, когда платёж только записывался, строки нет вовсе, и след остаётся в логе и в `getStarTransactions`, который библиотека не вызывает.

В 0.1.0 списка застрявших платежей не было: README предлагал искать их SQL-запросом и разбирать каждый вручную. В 0.2.0 эта часть появилась. `stuck_payments()` возвращает платежи в `error` и платежи в `processing`, у которых истекла аренда `payment_lease`. `accept_payment(charge_id)` помечает платёж принятым, когда товар вы выдали сами, а `reject_payment(charge_id, reason=...)` помечает его отклонённым и в той же транзакции ставит возврат в очередь, так же как `RejectPayment` из `on_paid`. Для возвратов, у которых кончились попытки, есть `failed_refunds()` со списком и `retry_refund(charge_id)`, который снова ставит возврат в очередь. Платёж, аренда которого ещё не истекла, закрыть нельзя: `on_paid` может ещё работать. Если два админа закрывают один платёж одновременно, получится у одного, второй получит `ValueError`. В README, в разделе [«Застрявшие платежи»](https://github.com/sinnercode228/aiogram-stars/blob/0c6625c82c10992b8c17446f9586cb77b4e2a3b8/README.ru.md#L477-L542), есть задача, которая присылает оба списка в админский чат, а в примере бота есть команды `/stuck`, `/accept`, `/reject` и `/retry`.

Но работа всё равно ручная. Сама библиотека `stuck_payments()` не вызывает, значит, его должен кто-то запускать, а что случилось с заказом, решает человек. Возврат в `failed`, который отправлять уже не нужно, так и остаётся в `failed_refunds()`, а возврат в `pending` отменить нельзя: методов для этого пока нет.

Ещё четыре ограничения:

- `on_refunded` и `on_refund_failed` вызываются один раз. Если процесс упал между записью результата и вызовом, уведомление пропадает, хотя статус в таблице верный.
- Таймаут pre-checkout покрывает только `check`. Время, за которое апдейт дошёл до бота, и сам запрос `answerPreCheckoutQuery` идут из тех же 10 секунд.
- Пауза из-за flood control действует на процесс, а не на базу.
- Тесты не ходят в Telegram. Они работают с поддельной сессией Bot API, ответы которой проходят через разбор ответов самого aiogram, поэтому ошибки приходят теми же классами исключений, что и в работе.

## Как попробовать

`pip install aiogram-stars` ставит библиотеку с `MemoryStorage`, а `pip install "aiogram-stars[sqlite]"` или `"aiogram-stars[postgres]"` добавляет SQLAlchemy с драйвером для `SqlAlchemyStorage`. Нужны Python 3.10+ и aiogram 3.15+. В репозитории есть README на русском, быстрый старт из README прогоняется в тестах, а `examples/shop_bot.py` — бот с `/buy`, `/refund` и `/paysupport`, а для админа ещё `/stuck`, `/accept`, `/reject` и `/retry`. Код и issues: [github.com/sinnercode228/aiogram-stars](https://github.com/sinnercode228/aiogram-stars).
