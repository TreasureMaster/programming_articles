# Запуск цикла событий Asyncio в другом потоке

Оригинал статьи: [https://www.linw1995.com/en/blog/Run-Asyncio-Event-Loop-in-another-thread/](https://www.linw1995.com/en/blog/Run-Asyncio-Event-Loop-in-another-thread/)

Дата публикации: 14 марта 2020

## Запуск цикла событий asyncio в другом потоке

При использовании веб-фреймворка, который не поддерживает запись параллельного кода с использованием синтаксиса **async/await**, вы хотите использовать **concurrent** для ускорения соединения с другой службой, например, подключение **Redis**, выполнение большого количества запросов и т. д. Итак, вот почему нам нужно запускать цикл событий **asyncio** в другом потоке.

## Создание потока, который запускает цикл событий asyncio навсегда

{% hint style="warning" %}
Внимание, этот пример полностью не протестирован.
{% endhint %}

```python
import asyncio
import threading

class AsyncioEventLoopThread(threading.Thread):
    def __init__(self, *args, loop=None, **kwargs):
        super().__init__(*args, **kwargs)
        self.loop = loop or asyncio.new_event_loop()
        self.running = False

    def run(self):
        self.running = True
        self.loop.run_forever()

    def run_coro(self, coro):
        return asyncio.run_coroutine_threadsafe(coro, loop=self.loop).result()

    def stop(self):
        self.loop.call_soon_threadsafe(self.loop.stop)
        self.join()
        self.running = False
```

Поскольку функция <mark style="color:red;">asyncio.run\_coroutine\_threadsafe</mark> возвращает объект <mark style="color:red;">concurrent.futures.Future</mark>, мы просто выполняем метод <mark style="color:red;">result</mark>, чтобы получить результат сопрограммы. Он будет ждать завершения сопрограммы.

Ниже приведен простой пример, демонстрирующий как его использовать.

```python
async def hello_world():
    print("hello world")

async def make_request():
    await asyncio.sleep(1)

thr = AsyncioEventLoopThread()
thr.start()
try:
    thr.run_coro(hello_world())
    thr.run_coro(make_request())
finally:
    thr.stop()
```

{% hint style="warning" %}
Внимание, не запускайте одну и ту же сопрограмму в двух разных циклах событий.
{% endhint %}

## Совместное использование объектов между сопрограммами

Вы должны унаследовать <mark style="color:red;">AsyncioEventLoopThread</mark> или просто изменить его, чтобы хранить объекты, которые необходимо совместно использовать между сопрограммами.

Используя <mark style="color:red;">contextvars</mark> для совместного использования объектов между сопрограммами, общие значения должны быть сохранены в <mark style="color:red;">contextvars.Context</mark> перед запуском сопрограмм.

```python
import contextvars
import aiohttp

var_session = contextvars.ContextVar('session')

class FetcherThread(AsyncioEventLoopThread):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self._session = None
        self._event_session_created = threading.Event()

    def run_coro(self, coro):
        self._event_session_created.wait()
        var_session.set(self.session)
        return super().run_coro(coro)

    async def _create_session(self):
        self.session = aiohttp.ClientSession()

    async def _close_session(self):
        await self.session.close()

    def run(self):
        fut = asyncio.run_coroutine_threadsafe(self._create_session(), loop=self.loop)
        fut.add_done_callback(lambda _: self._event_session_created.set())
        super().run()

    def stop(self):
        self.run_coro(self._close_session())
        super().stop()
```

Ниже приведен простой пример получения исходного кода веб-сайта <mark style="color:blue;">github.com</mark>.

```python
async def make_request():
    session = var_session.get()
    async with session.get("https://github.com") as resp:
        resp.raise_for_status()
        return await resp.text()


thr = FetcherThread()
thr.start()
try:
    text = thr.run_coro(make_request())
    print(text)
finally:
    thr.stop()
```

## Ссылки

* [Concurrency and Multithreading | Developing with asyncio](https://docs.python.org/3/library/asyncio-dev.html#concurrency-and-multithreading)
* [asyncio support | contextvars — Context Variables](https://docs.python.org/3/library/contextvars.html#asyncio-support)
* [Event Objects | threading — Thread-based parallelism](https://docs.python.org/3/library/threading.html#event-objects)
