# Учебное пособие по Python async/await

Асинхронное программирование набирает обороты в последние несколько лет, и не зря. Хотя это может быть сложнее, чем традиционный линейный стиль, он также намного эффективнее.

Например, вместо ожидания завершения HTTP-запроса перед продолжением выполнения с помощью асинхронных сопрограмм Python вы можете отправить запрос и выполнить другую работу, ожидающую в очереди, пока ожидает завершения HTTP-запроса. Чтобы понять правильную логику, может потребоваться немного больше размышлений, но вы сможете справиться с гораздо большей работой с меньшими ресурсами.

Даже тогда синтаксис и выполнение асинхронных функций в таких языках, как Python, на самом деле не так уж и сложны. Другое дело, с JavaScript, но Python, похоже, выполняет его довольно хорошо.

Асинхронность, кажется, главная причина того, почему Node.js так популярен для серверного программирования. Большая часть кода, который мы пишем, особенно в тяжелых приложениях ввода-вывода, таких как веб-сайты, зависит от внешних ресурсов. Это может быть что угодно, от удаленного вызова базы данных до POSTing в службу REST. Как только вы запрашиваете какой-либо из этих ресурсов, ваш код ждет, ничего не делая.

При асинхронном программировании вы позволяете своему коду обрабатывать другие задачи, ожидая ответа от других ресурсов.

## Сопрограммы

Асинхронная функция в Python обычно называется сопрограммой, которая представляет собой просто функцию, использующую ключевое слово <mark style="color:red;">**async**</mark> или декоратор <mark style="color:red;">@asyncio.coroutine</mark>. Любая из приведенных ниже функций будет работать как сопрограмма и фактически эквивалентна по типу:

```python
import asyncio

async def ping_server(ip):
    pass

@asyncio.coroutine
def load_file(path):
    pass
```

Это специальные функции, которые при вызове возвращают объекты сопрограмм. Если вы знакомы с обещаниями JavaScript, то можете думать об этом возвращаемом объекте почти как об обещании. Вызов любого из них на самом деле не запускает их, но вместо этого возвращается объект [сопрограммы](https://docs.python.org/3/library/asyncio-task.html#coroutines), который затем может быть передан в цикл событий для последующего выполнения.

На случай, если вам когда-нибудь понадобится определить, является ли функция сопрограммой или нет, <mark style="color:red;">**asyncio**</mark> предоставляет метод <mark style="color:red;">asyncio.iscoroutinefunction(func)</mark>, который делает именно это за вас. Или, если вам нужно определить, является ли объект, возвращаемый функцией, объектом сопрограммы, вы можете вместо этого использовать <mark style="color:red;">asyncio.iscoroutine(obj)</mark>.

## Yield from <a href="#yieldfrom" id="yieldfrom"></a>

Есть несколько способов вызвать сопрограмму, один из которых - метод <mark style="color:red;">**yield from**</mark>. Это было введено в Python 3.3 и было улучшено в Python 3.5 в форме <mark style="color:red;">**async/await**</mark> (о которой мы поговорим позже).

Выражение <mark style="color:red;">**yield from**</mark> может использоваться следующим образом:

```python
import asyncio

@asyncio.coroutine
def get_json(client, url):
    file_content = yield from load_file('/Users/scott/data.txt')
```

Как видите, <mark style="color:red;">**yield from**</mark> используется в функции, украшенной <mark style="color:red;">@asyncio.coroutine</mark>. Если бы вы попытались использовать <mark style="color:red;">**yield from**</mark> вне этой функции, то получили бы такую ошибку от Python:

```bash
  File "main.py", line 1
    file_content = yield from load_file('/Users/scott/data.txt')
                  ^
SyntaxError: 'yield' outside function
```

Чтобы использовать этот синтаксис, он должен находиться в другой функции (обычно с декоратором сопрограммы).

## Async/await <a href="#asyncawait" id="asyncawait"></a>

Более новый и понятный синтаксис заключается в использовании ключевых слов <mark style="color:red;">**async/await**</mark>. Представленный в Python 3.5, <mark style="color:red;">**async**</mark> используется для объявления функции как сопрограммы, подобно тому, как это делает декоратор <mark style="color:red;">@asyncio.coroutine</mark>. Его можно применить к функции, поместив его перед определением:

```python
async def ping_server(ip):
    # ping code here...
```

Чтобы вызвать эту функцию, мы используем <mark style="color:red;">**await**</mark> вместо <mark style="color:red;">**yield from**</mark>, но во многом таким же образом:

```python
async def ping_local():
    return await ping_server('192.168.1.1')
```

Опять же, как и <mark style="color:red;">**yield from**</mark>, вы не можете использовать это вне другой сопрограммы, иначе вы получите синтаксическую ошибку.

В Python 3.5 поддерживаются оба способа вызова сопрограмм, но способ <mark style="color:red;">**async/await**</mark> должен быть основным синтаксисом.

## Запуск цикла событий

Ни один из описанных выше сопрограмм не будет иметь значения (или работать), если вы не знаете, как стартовать и запускать [цикл событий](https://docs.python.org/3/library/asyncio-eventloop.html). Цикл событий - это центральная точка выполнения асинхронных функций, поэтому, когда вы действительно хотите выполнить сопрограмму, это то, что вы будете использовать.

Цикл событий предоставляет вам несколько функций:

* Регистрация, выполнение и отмена отложенных вызовов (асинхронные функции)
* Создание клиентских и серверных транспортов для связи
* Создание подпроцессов и транспортов для связи с другой программой
* Делегирование вызовов функций пулу потоков

Хотя на самом деле существует довольно много конфигураций и типов циклов событий, которые вы можете использовать, большинству программ, которые вы пишете, просто нужно будет использовать что-то вроде этого для планирования функции:

```python
import asyncio

async def speak_async():
    print('OMG asynchronicity!')

loop = asyncio.get_event_loop()
loop.run_until_complete(speak_async())
loop.close()
```

Последние три строки - это то, что нас здесь интересует. Он начинается с получения цикла событий по умолчанию (<mark style="color:red;">asyncio.get\_event\_loop ()</mark>), планирования и выполнения задачи <mark style="color:red;">**async**</mark>, а затем закрытия цикла, когда цикл завершен.

Функция <mark style="color:red;">loop.run\_until\_complete ()</mark> фактически блокирует, поэтому она не вернется, пока не будут выполнены все асинхронные методы. Поскольку мы выполняем это только в одном потоке, он не может двигаться вперед, пока цикл выполняется.

Теперь вы можете подумать, что это не очень полезно, поскольку мы все равно блокируем цикл событий (вместо только вызовов ввода-вывода), но представьте, что вы обертываете всю свою программу в асинхронной функции, которая затем позволит вам запускать много асинхронных запросы одновременно, как на веб-сервере.

Вы даже можете прервать цикл обработки событий в отдельный поток, позволяя ему обрабатывать все длинные запросы ввода-вывода, в то время как основной поток обрабатывает логику программы или пользовательский интерфейс.

## Пример

Хорошо, давайте посмотрим на более крупный пример, который мы действительно можем запустить. Следующий код представляет собой довольно простую асинхронную программу, которая извлекает JSON из Reddit, анализирует JSON и распечатывает самые популярные сообщения дня из <mark style="color:red;">/r/python</mark>, <mark style="color:red;">/r/programming</mark> и <mark style="color:red;">/r/compsci</mark>.

Первый показанный метод, <mark style="color:red;">get\_json ()</mark>, вызывается <mark style="color:red;">get\_reddit\_top ()</mark> и просто создает HTTP-запрос GET на соответствующий URL-адрес Reddit. Когда это вызывается с помощью <mark style="color:red;">**await**</mark>, цикл событий может продолжаться и обслуживать другие сопрограммы, ожидая ответа HTTP. Как только это произойдет, JSON возвращается в <mark style="color:red;">get\_reddit\_top ()</mark>, анализируется и распечатывается.

```python
import signal
import sys
import asyncio
import aiohttp
import json

loop = asyncio.get_event_loop()
client = aiohttp.ClientSession(loop=loop)

async def get_json(client, url):
    async with client.get(url) as response:
        assert response.status == 200
        return await response.read()

async def get_reddit_top(subreddit, client):
    data1 = await get_json(client, 'https://www.reddit.com/r/' + subreddit + '/top.json?sort=top&t=day&limit=5')

    j = json.loads(data1.decode('utf-8'))
    for i in j['data']['children']:
        score = i['data']['score']
        title = i['data']['title']
        link = i['data']['url']
        print(str(score) + ': ' + title + ' (' + link + ')')

    print('DONE:', subreddit + '\n')

def signal_handler(signal, frame):
    loop.stop()
    client.close()
    sys.exit(0)

signal.signal(signal.SIGINT, signal_handler)

asyncio.ensure_future(get_reddit_top('python', client))
asyncio.ensure_future(get_reddit_top('programming', client))
asyncio.ensure_future(get_reddit_top('compsci', client))
loop.run_forever()
```

Это немного отличается от примера кода, который мы показали ранее. Чтобы запустить несколько сопрограмм в цикле событий, мы используем <mark style="color:red;">asyncio.ensure\_future ()</mark>, а затем запускаем цикл навсегда, чтобы обработать все.

Чтобы запустить это, вам нужно сначала установить <mark style="color:red;">**aiohttp**</mark>, что вы можете сделать с помощью PIP:

```bash
$ pip install aiohttp
```

Теперь просто убедитесь, что вы запускаете его с Python 3.5 или выше, и вы должны получить следующий результат:

```bash
$ python main.py
46: Python async/await Tutorial (http://stackabuse.com/python-async-await-tutorial/)
16: Using game theory (and Python) to explain the dilemma of exchanging gifts. Turns out: giving a gift probably feels better than receiving one... (http://vknight.org/unpeudemath/code/2015/12/15/The-Prisoners-Dilemma-of-Christmas-Gifts/)
56: Which version of Python do you use? (This is a poll to compare the popularity of Python 2 vs. Python 3) (http://strawpoll.me/6299023)
DONE: python

71: The Semantics of Version Control - Wouter Swierstra (http://www.staff.science.uu.nl/~swier004/Talks/vc-semantics-15.pdf)
25: Favorite non-textbook CS books (https://www.reddit.com/r/compsci/comments/3xag9e/favorite_nontextbook_cs_books/)
13: CompSci Weekend SuperThread (December 18, 2015) (https://www.reddit.com/r/compsci/comments/3xacch/compsci_weekend_superthread_december_18_2015/)
DONE: compsci

1752: 684.8 TB of data is up for grabs due to publicly exposed MongoDB databases (https://blog.shodan.io/its-still-the-data-stupid/)
773: Instagram's Million Dollar Bug? (http://exfiltrated.com/research-Instagram-RCE.php)
387: Amazingly simple explanation of Diffie-Hellman. His channel has tons of amazing videos and only a few views :( thought I would share! (https://www.youtube.com/watch?v=Afyqwc96M1Y)
DONE: programming
```

Обратите внимание, что если вы запустите это несколько раз, порядок, в котором печатаются данные субреддита, изменится. Это связано с тем, что каждый из наших вызовов освобождает (передает) контроль над потоком, позволяя выполнить другой HTTP-вызов. То, что вернется первым, распечатывается первым.

## Заключение

Хотя встроенные в Python асинхронные функции не так удобны, как JavaScript, это не значит, что вы не можете использовать их для интересных и эффективных приложений. Просто потратьте 30 минут, чтобы изучить его все входы и выходы, и вы гораздо лучше поймете, как вы можете интегрировать это в свои собственные приложения.
