# Самый быстрый способ загрузки данных в PostgreSQL с помощью Python

#### От двух минут до менее чем полсекунды!

{% hint style="info" %}
Оригинал статьи: [Fastest Way to Load Data Into PostgreSQL Using Python](https://hakibenita.com/fast-load-data-python-postgresql)

Дата публикации: 9 июля 2019

Автор: [Haki Benita](https://hakibenita.com/)
{% endhint %}

Нам, как прославленным сборщикам данных, часто приходится загружать данные, полученные из удаленного источника, в наши системы. Если нам повезет, данные сериализуются в формате JSON или YAML. Когда нам везет меньше, мы получаем электронную таблицу Excel или файл CSV, который всегда каким-то образом поврежден, и это невозможно объяснить.

Данные крупных компаний или старых систем каким-то образом всегда кодируются странным образом, и системные администраторы всегда думают, что делают нам одолжение, архивируя файлы (пожалуйста, gzip) или разбивая их на более мелкие файлы со случайными именами.

Современные сервисы могут предоставлять достойный API, но чаще всего нам нужно получить файл с FTP, SFTP, S3 или какого-либо собственного хранилища, которое работает только в Windows.

_**В этой статье мы рассмотрим лучший способ импорта беспорядочных данных из удаленного источника в PostgreSQL.**_

Чтобы обеспечить реальное и работоспособное решение, мы установили следующие основные роли:

1. Данные извлекаются из удаленного источника.
2. Данные загрязнены и их необходимо преобразовать.
3. Данные большие.

## Оглавление

* [Установка: пивоварня](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#ustanovka-pivovarnya)
  * [Данные](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#dannye)
  * [Получение данных](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#poluchenie-dannykh)
  * [Создать таблицу в базе данных](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#sozdat-tablicu-v-baze-dannykh)
* [Метрики](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#metriki)
  * [Измерение времени](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#izmerenie-vremeni)
  * [Измерение памяти](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#izmerenie-pamyati)
  * [Декоратор profile](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#dekorator-profile)
* [Проверка](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#proverka)
  * [Вставка строк одна за другой (execute)](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#vstavka-strok-odna-za-drugoi-execute)
  * [Выполнить много (executemany)](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#vypolnit-mnogo-executemany)
  * [Выполнить много из итератора (executemany)](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#vypolnit-mnogo-iz-iteratora-executemany)
  * [Выполнить пакетно (execute\_batch)](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#vypolnit-paketno-execute\_batch)
  * [Выполнить пакетно из итератора (execute\_batch)](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#vypolnit-paketno-iz-iteratora-execute\_batch)
  * [Выполнение пакетно из итератора с размером страницы (execute\_batch)](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#vypolnenie-paketno-iz-iteratora-s-razmerom-stranicy-execute\_batch)
  * [Выполнение с значениями (execute\_values)](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#vypolnenie-s-znacheniyami-execute\_values)
  * [Выполнение с значениями из итератора (execute\_values)](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#vypolnenie-s-znacheniyami-iz-iteratora-execute\_values)
  * [Выполнение с значениями из итератора с размером страницы (execute\_values)](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#vypolnenie-s-znacheniyami-iz-iteratora-s-razmerom-stranicy-execute\_values)
  * [Копировать (copy\_from)](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#kopirovat-copy\_from)
  * [Копирование данных из строкового итератора (copy\_from)](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#kopirovanie-dannykh-iz-strokovogo-iteratora-copy\_from)
  * [Копирование данных из строкового итератора с размером буфера (copy\_from)](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#kopirovanie-dannykh-iz-strokovogo-iteratora-s-razmerom-bufera-copy\_from)
* [Сводка результатов](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#svodka-rezultatov)
* [Краткое содержание](samyi-bystryi-sposob-zagruzki-dannykh-v-postgresql-s-pomoshyu-python.md#kratkoe-soderzhanie)



## Установка: пивоварня

Я нашел [отличный общедоступный API для пива](https://punkapi.com/documentation/v2), поэтому мы собираемся импортировать данные в таблицу пива в базе данных.

### Данные

Одиночный запрос на пиво из API выглядит так:

```bash
$ curl https://api.punkapi.com/v2/beers/?per_page=1&page=1
[
    {
        "id": 1,
        "name": "Buzz",
        "tagline": "A Real Bitter Experience.",
        "first_brewed": "09/2007",
        "description": "A light, crisp and bitter IPA ...",
        "image_url": "https://images.punkapi.com/v2/keg.png",
        "abv": 4.5,
        "ibu": 60,
        "target_fg": 1010,
        "target_og": 1044,
        "ebc": 20,
        "srm": 10,
        "ph": 4.4,
        "attenuation_level": 75,
        "volume": {
            "value": 20,
            "unit": "litres"
        },
        "contributed_by": "Sam Mason <samjbmason>"
        "brewers_tips": "The earthy and floral aromas from...",
        "boil_volume": {},
        "method": {},
        "ingredients": {},
        "food_pairing": [],
    }
]
```

Для краткости я обрезал вывод, но здесь много информации о пиве. В этой статье мы хотим импортировать все поля перед brewers\_tips в таблицу базы данных.

Поле volume является вложенным. Мы хотим извлечь из поля только значение value и сохранить его в поле таблицы с именем volume.

```python
volume = beer['volume']['value']
```

Поле first\_brewed содержит только год и месяц, а в некоторых случаях только год. Мы хотим преобразовать значение в действительную дату. Например, значение 09/2007 будет преобразовано в дату 01.09.2007. Значение 2006 будет преобразовано в дату 01 января 2016 г.

Давайте напишем простую функцию для преобразования текстового значения в поле в datetime.date Python:

```python
import datetime

def parse_first_brewed(text: str) -> datetime.date:
    parts = text.split('/')
    if len(parts) == 2:
        return datetime.date(int(parts[1]), int(parts[0]), 1)
    elif len(parts) == 1:
        return datetime.date(int(parts[0]), 1, 1)
    else:
        assert False, 'Unknown date format'
```

Давайте быстро убедимся, что это работает:

```python
>>> parse_first_brewed('09/2007')
datetime.date(2007, 9, 1)

>>> parse_first_brewed('2006')
datetime.date(2006, 1, 1)
```

В реальной жизни трансформации могут быть гораздо сложнее. Но для наших целей этого более чем достаточно.

### Получение данных

API предоставляет постраничные результаты. Чтобы инкапсулировать подкачку, мы создаем генератор, который выдает пиво одно за другим:

```python
from typing import Iterator, Dict, Any
from urllib.parse import urlencode
import requests


def iter_beers_from_api(page_size: int = 5) -> Iterator[Dict[str, Any]]:
    session = requests.Session()
    page = 1
    while True:
        response = session.get('https://api.punkapi.com/v2/beers?' + urlencode({
            'page': page,
            'per_page': page_size
        }))
        response.raise_for_status()

        data = response.json()
        if not data:
            break

        yield from data

        page += 1
```

И чтобы использовать функцию генератора, мы вызываем и повторяем ее:

```python
>>> beers = iter_beers_from_api()
>>> next(beers)
{'id': 1,
 'name': 'Buzz',
 'tagline': 'A Real Bitter Experience.',
 'first_brewed': '09/2007',
 'description': 'A light, crisp and bitter IPA brewed...',
 'image_url': 'https://images.punkapi.com/v2/keg.png',
 'abv': 4.5,
 'ibu': 60,
 'target_fg': 1010,
...
}
>>> next(beers)
{'id': 2,
 'name': 'Trashy Blonde',
 'tagline': "You Know You Shouldn't",
 'first_brewed': '04/2008',
 'description': 'A titillating, ...',
 'image_url': 'https://images.punkapi.com/v2/2.png',
 'abv': 4.1,
 'ibu': 41.5,
```

Вы заметите, что первый результат каждой страницы занимает немного больше времени. Это связано с тем, что он выполняет сетевой запрос для получения страницы.

### Создать таблицу в базе данных

Следующим шагом будет создание таблицы в базе данных для импорта данных.

Создайте базу данных:

```bash
$ createdb -O haki testload
```

Измените haki в примере на своего локального пользователя.

Для подключения Python к базе данных PostgreSQL мы используем [psycopg](http://initd.org/psycopg/):

```bash
$ python -m pip install psycopg2
```

Используя psycopg, создайте соединение с базой данных:

```python
import psycopg2

connection = psycopg2.connect(
    host="localhost",
    database="testload",
    user="haki",
    password=None,
)
connection.autocommit = True
```

Мы устанавливаем `autocommit=True`, чтобы каждая выполняемая нами команда вступала в силу немедленно. Для целей данной статьи это нормально.

Теперь, когда у нас есть соединение, мы можем написать функцию для создания таблицы:

```python
def create_staging_table(cursor) -> None:
    cursor.execute("""
        DROP TABLE IF EXISTS staging_beers;
        CREATE UNLOGGED TABLE staging_beers (
            id                  INTEGER,
            name                TEXT,
            tagline             TEXT,
            first_brewed        DATE,
            description         TEXT,
            image_url           TEXT,
            abv                 DECIMAL,
            ibu                 DECIMAL,
            target_fg           DECIMAL,
            target_og           DECIMAL,
            ebc                 DECIMAL,
            srm                 DECIMAL,
            ph                  DECIMAL,
            attenuation_level   DECIMAL,
            brewers_tips        TEXT,
            contributed_by      TEXT,
            volume              INTEGER
        );
    """)
```

Функция получает курсор и создает нерегистрируемую таблицу с именем staging\_beers.

{% hint style="info" %}
**UNLOGGED TABLE**

Данные, записываемые в [unlogged table](https://www.postgresql.org/docs/current/sql-createtable.html#id-1.9.3.85.6), не будут регистрироваться в журнале упреждающей записи (WAL), что делает его идеальным для промежуточных таблиц. Обратите внимание, что таблицы UNLOGGED не будут восстановлены в случае сбоя и не будут реплицироваться.
{% endhint %}

Используя соединение, которое мы создали ранее, функция применяется следующим образом:

```python
>>> with connection.cursor() as cursor:
>>>     create_staging_table(cursor)
```

Теперь мы готовы перейти к следующей части.

## Метрики

В этой статье нас интересуют две основные метрики: время и память.

### Измерение времени

Для измерения времени для каждого метода мы используем встроенный [модуль time](https://docs.python.org/3/library/time.html):

```python
>>> import time
>>> start = time.perf_counter()
>>> time.sleep(1) # do work
>>> elapsed = time.perf_counter() - start
>>> print(f'Time {elapsed:0.4}')
Time 1.001
```

Функция [perf\_counter](https://docs.python.org/3/library/time.html#time.perf\_counter) предоставляет часам максимально возможное разрешение, что делает ее идеальной для наших целей.

### Измерение памяти

Для измерения потребления памяти мы воспользуемся пакетом [memory-profiler](https://pypi.org/project/memory-profiler/).

```bash
$ python -m pip install memory-profiler
```

Этот пакет обеспечивает использование памяти и дополнительное использование памяти для каждой строки кода. Это очень полезно при оптимизации памяти. Для иллюстрации это пример, представленный в PyPI:

```bash
$ python -m memory_profiler example.py

Line #    Mem usage  Increment   Line Contents
==============================================
     3                           @profile
     4      5.97 MB    0.00 MB   def my_func():
     5     13.61 MB    7.64 MB       a = [1] * (10 ** 6)
     6    166.20 MB  152.59 MB       b = [2] * (2 * 10 ** 7)
     7     13.61 MB -152.59 MB       del b
     8     13.61 MB    0.00 MB       return a
```

Интересная часть — это столбец Increment, который показывает дополнительную память, выделенную кодом в каждой строке.

В этой статье нас интересует пиковая память, используемая функцией. Пиковый объем памяти — это разница между начальным значением столбца "Mem usage" и максимальным значением (также известным как "high watermark").

Чтобы получить список "Mem usage", мы используем функцию memory\_usage из memory\_profiler:

```python
>>> from memory_profiler import memory_usage
>>> mem, retval = memory_usage((fn, args, kwargs), retval=True, interval=1e-7)
```

При таком использовании функция memory\_usage выполняет функцию fn с предоставленными аргументами args и kwargs, а также запускает другой процесс в фоновом режиме для мониторинга использования памяти каждый inetrval секунд.

Для очень быстрых операций функция fn может выполняться более одного раза. Установив интервал interval  [ниже 1e-6](https://github.com/pythonprofilers/memory\_profiler/blob/0.55/memory\_profiler.py#L350), мы заставляем его выполняться только один раз.

Аргумент retval сообщает функции, что она должна вернуть результат fn.

### Декоратор profile

Чтобы собрать все это вместе, мы создаем следующий декоратор для измерения и отчета о времени и памяти:

```python
import time
from functools import wraps
from memory_profiler import memory_usage

def profile(fn):
    @wraps(fn)
    def inner(*args, **kwargs):
        fn_kwargs_str = ', '.join(f'{k}={v}' for k, v in kwargs.items())
        print(f'\n{fn.__name__}({fn_kwargs_str})')

        # Measure time
        t = time.perf_counter()
        retval = fn(*args, **kwargs)
        elapsed = time.perf_counter() - t
        print(f'Time   {elapsed:0.4}')

        # Measure memory
        mem, retval = memory_usage(
            (fn, args, kwargs), retval=True, timeout=200, interval=1e-7
        )

        print(f'Memory {max(mem) - min(mem)}')
        return retval

    return inner
```

Чтобы исключить взаимное влияние таймингов на память и наоборот, мы выполняем функцию дважды. Во-первых, чтобы рассчитать время, во-вторых, чтобы измерить использование памяти.

Декоратор выведет имя функции и все аргументы ключевого слова, а также сообщит об использованном времени и памяти:

```python
>>> @profile
>>> def work(n):
>>>     for i in range(n):
>>>         2 ** n

>>> work(10)
work()
Time   0.06269
Memory 0.0

>>> work(n=10000)
work(n=10000)
Time   0.3865
Memory 0.0234375
```

Печатаются только аргументы ключевых слов. Это сделано намеренно, мы собираемся использовать это в параметризованных тестах.

## Проверка

На момент написания API пива содержит только 325 сортов пива. Чтобы работать с большим набором данных, мы дублируем его 100 раз и сохраняем в памяти. Результирующий набор данных содержит 32 500 сортов пива:

```python
>>> beers = list(iter_beers_from_api()) * 100
>>> len(beers)
32,500
```

Чтобы имитировать удаленный API, наши функции будут принимать итераторы, аналогичные возвращаемому значению iter\_beers\_from\_api:

```python
def process(beers: Iterator[Dict[str, Any]])) -> None:
    # Process beers...
```

Для тестирования мы собираемся импортировать данные о пиве в базу данных. Чтобы устранить внешние влияния, такие как сеть, мы заранее получаем данные из API и обслуживаем их локально.

Чтобы получить точное время, мы «подделываем» удаленный API:

```python
>>> beers = list(iter_beers_from_api()) * 100
>>> process(beers)
```

В реальной жизненной ситуации вы бы использовали функцию iter\_beers\_from\_api напрямую:

```python
>>> process(iter_beers_from_api())
```

Теперь мы готовы начать!

### Вставка строк одна за другой (execute)

Чтобы установить базовый уровень, мы начнем с самого простого подхода - вставляем строки одну за другой:

```python
@profile
def insert_one_by_one(connection, beers: Iterator[Dict[str, Any]]) -> None:
    with connection.cursor() as cursor:
        create_staging_table(cursor)
        for beer in beers:
            cursor.execute("""
                INSERT INTO staging_beers VALUES (
                    %(id)s,
                    %(name)s,
                    %(tagline)s,
                    %(first_brewed)s,
                    %(description)s,
                    %(image_url)s,
                    %(abv)s,
                    %(ibu)s,
                    %(target_fg)s,
                    %(target_og)s,
                    %(ebc)s,
                    %(srm)s,
                    %(ph)s,
                    %(attenuation_level)s,
                    %(brewers_tips)s,
                    %(contributed_by)s,
                    %(volume)s
                );
            """, {
                **beer,
                'first_brewed': parse_first_brewed(beer['first_brewed']),
                'volume': beer['volume']['value'],
            })
```

Обратите внимание: при переборе пива мы преобразуем first\_brewed в datetime.date и извлекаем значение объема из вложенного поля объема.

Запуск этой функции дает следующий результат:

```python
>>> insert_one_by_one(connection, beers)
insert_one_by_one()
Time   128.8
Memory 0.08203125
```

Для импорта 32 тыс. строк функции потребовалось 129 секунд. Профилировщик памяти показывает, что функция потребляла очень мало памяти.

Интуитивно понятно, что вставка строк одна за другой кажется не очень эффективной. Должно быть, постоянное переключение контекста между программой и базой данных замедляет ее работу.

### Выполнить много (executemany)

Psycopg2 предоставляет возможность вставлять множество строк одновременно с помощью выполнения [executemany](http://initd.org/psycopg/docs/cursor.html#cursor.executemany). Из документов:

> Выполнить операцию базы данных (запрос или команду) для всех кортежей параметров или сопоставлений, найденных в последовательности vars\_list.

Звучит многообещающе!

Давайте попробуем импортировать данные с помощью executemany:

```python
@profile
def insert_executemany(connection, beers: Iterator[Dict[str, Any]]) -> None:
    with connection.cursor() as cursor:
        create_staging_table(cursor)

        all_beers = [{
            **beer,
            'first_brewed': parse_first_brewed(beer['first_brewed']),
            'volume': beer['volume']['value'],
        } for beer in beers]

        cursor.executemany("""
            INSERT INTO staging_beers VALUES (
                %(id)s,
                %(name)s,
                %(tagline)s,
                %(first_brewed)s,
                %(description)s,
                %(image_url)s,
                %(abv)s,
                %(ibu)s,
                %(target_fg)s,
                %(target_og)s,
                %(ebc)s,
                %(srm)s,
                %(ph)s,
                %(attenuation_level)s,
                %(brewers_tips)s,
                %(contributed_by)s,
                %(volume)s
            );
        """, all_beers)
```

Функция внешне очень похожа на предыдущую функцию, и преобразования такие же. Основное отличие здесь в том, что мы сначала преобразуем все данные в памяти и только потом импортируем их в базу данных.

Запуск этой функции дает следующий результат:

```python
>>> insert_executemany(connection, beers)
insert_executemany()
Time   124.7
Memory 2.765625
```

Это разочаровывает. Время стало немного лучше, но теперь функция потребляет 2,7 МБ памяти.

Чтобы оценить использование памяти, файл JSON, содержащий только импортируемые нами данные, весит на диске 25 МБ. Учитывая пропорции, использование этого метода для импорта файла размером 1 ГБ потребует 110 МБ памяти.

### Выполнить много из итератора (executemany)

Предыдущий метод потреблял много памяти, поскольку преобразованные данные сохранялись в памяти до обработки psycopg.

Давайте посмотрим, можем ли мы использовать итератор, чтобы избежать хранения данных в памяти:

```python
@profile
def insert_executemany_iterator(connection, beers: Iterator[Dict[str, Any]]) -> None:
    with connection.cursor() as cursor:
        create_staging_table(cursor)
        cursor.executemany("""
            INSERT INTO staging_beers VALUES (
                %(id)s,
                %(name)s,
                %(tagline)s,
                %(first_brewed)s,
                %(description)s,
                %(image_url)s,
                %(abv)s,
                %(ibu)s,
                %(target_fg)s,
                %(target_og)s,
                %(ebc)s,
                %(srm)s,
                %(ph)s,
                %(attenuation_level)s,
                %(brewers_tips)s,
                %(contributed_by)s,
                %(volume)s
            );
        """, ({
            **beer,
            'first_brewed': parse_first_brewed(beer['first_brewed']),
            'volume': beer['volume']['value'],
        } for beer in beers))
```

Разница здесь в том, что преобразованные данные «передаются в поток» в метод выполнения с помощью итератора.

Эта функция дает следующий результат:

```python
>>> insert_executemany_iterator(connection, beers)
insert_executemany_iterator()
Time   129.3
Memory 0.0
```

Наше «потоковое» решение сработало как положено, и нам удалось свести объем памяти к нулю. Однако сроки остаются примерно такими же, даже по сравнению с методом «один за другим».

### Выполнить пакетно (execute\_batch)

В документации psycopg в разделе ["fast execution helpers"](http://initd.org/psycopg/docs/extras.html#fast-execution-helpers) есть очень интересное примечание о выполнении executemany:

> Текущая реализация метода выполнения executemany() (очень мягко говоря) не особенно эффективна. Эти функции можно использовать для ускорения повторного выполнения оператора с набором параметров. Уменьшив количество обращений к серверу, производительность может быть на несколько порядков выше, чем при использовании метода выполнения executemany().

Значит, мы всё время делали всё неправильно!

Функция чуть ниже этого раздела — [execute\_batch](http://initd.org/psycopg/docs/extras.html#psycopg2.extras.execute\_batch):

> Выполняйте группы операторов за меньшее количество обращений к серверу.

Давайте реализуем функцию загрузки с помощью execute\_batch:

```python
import psycopg2.extras

@profile
def insert_execute_batch(connection, beers: Iterator[Dict[str, Any]]) -> None:
    with connection.cursor() as cursor:
        create_staging_table(cursor)

        all_beers = [{
            **beer,
            'first_brewed': parse_first_brewed(beer['first_brewed']),
            'volume': beer['volume']['value'],
        } for beer in beers]

        psycopg2.extras.execute_batch(cursor, """
            INSERT INTO staging_beers VALUES (
                %(id)s,
                %(name)s,
                %(tagline)s,
                %(first_brewed)s,
                %(description)s,
                %(image_url)s,
                %(abv)s,
                %(ibu)s,
                %(target_fg)s,
                %(target_og)s,
                %(ebc)s,
                %(srm)s,
                %(ph)s,
                %(attenuation_level)s,
                %(brewers_tips)s,
                %(contributed_by)s,
                %(volume)s
            );
        """, all_beers)
```

Выполнение функции:

```python
>>> insert_execute_batch(connection, beers)
insert_execute_batch()
Time   3.917
Memory 2.50390625
```

Ух ты! Это огромный скачок. Функция завершилась менее чем за 4 секунды. Это примерно в 33 раза быстрее, чем 129 секунд, с которых мы начали.

### Выполнить пакетно из итератора (execute\_batch)

Функция execute\_batch использовала меньше памяти, чем executemany для тех же данных. Давайте попробуем освободить память, «передавая» данные в execute\_batch с помощью итератора:

```python
@profile
def insert_execute_batch_iterator(connection, beers: Iterator[Dict[str, Any]]) -> None:
    with connection.cursor() as cursor:
        create_staging_table(cursor)

        iter_beers = ({
            **beer,
            'first_brewed': parse_first_brewed(beer['first_brewed']),
            'volume': beer['volume']['value'],
        } for beer in beers)

        psycopg2.extras.execute_batch(cursor, """
            INSERT INTO staging_beers VALUES (
                %(id)s,
                %(name)s,
                %(tagline)s,
                %(first_brewed)s,
                %(description)s,
                %(image_url)s,
                %(abv)s,
                %(ibu)s,
                %(target_fg)s,
                %(target_og)s,
                %(ebc)s,
                %(srm)s,
                %(ph)s,
                %(attenuation_level)s,
                %(brewers_tips)s,
                %(contributed_by)s,
                %(volume)s
            );
        """, iter_beers)
```

Выполнение функции

```python
>>> insert_execute_batch_iterator(connection, beers)
insert_execute_batch_iterator()
Time   4.333
Memory 0.2265625
```

Получили примерно то же время, но с меньшим объемом памяти.

### Выполнение пакетно из итератора с размером страницы (execute\_batch)

Когда я читал [документацию по execute\_batch](http://initd.org/psycopg/docs/extras.html#psycopg2.extras.execute\_batch), мне в глаза бросился аргумент page\_size:

> page\_size – максимальное количество элементов списка аргументов, которые можно включить в каждый оператор. Если элементов больше, функция выполнит более одного оператора.

Ранее в документации указывалось, что функция работает лучше, поскольку она меньше обращается к базе данных. В этом случае больший размер страницы должен уменьшить количество обращений туда и обратно и привести к более быстрому времени загрузки.

Давайте добавим в нашу функцию аргумент размера страницы, чтобы мы могли поэкспериментировать:

```python
@profile
def insert_execute_batch_iterator(
    connection,
    beers: Iterator[Dict[str, Any]],
    page_size: int = 100,
) -> None:
    with connection.cursor() as cursor:
        create_staging_table(cursor)

        iter_beers = ({
            **beer,
            'first_brewed': parse_first_brewed(beer['first_brewed']),
            'volume': beer['volume']['value'],
        } for beer in beers)

        psycopg2.extras.execute_batch(cursor, """
            INSERT INTO staging_beers VALUES (
                %(id)s,
                %(name)s,
                %(tagline)s,
                %(first_brewed)s,
                %(description)s,
                %(image_url)s,
                %(abv)s,
                %(ibu)s,
                %(target_fg)s,
                %(target_og)s,
                %(ebc)s,
                %(srm)s,
                %(ph)s,
                %(attenuation_level)s,
                %(brewers_tips)s,
                %(contributed_by)s,
                %(volume)s
            );
        """, iter_beers, page_size=page_size)
```

Размер страницы по умолчанию — 100. Давайте сравним разные значения и результаты:

```python
>>> insert_execute_batch_iterator(connection, iter(beers), page_size=1)
insert_execute_batch_iterator(page_size=1)
Time   130.2
Memory 0.0

>>> insert_execute_batch_iterator(connection, iter(beers), page_size=100)
insert_execute_batch_iterator(page_size=100)
Time   4.333
Memory 0.0

>>> insert_execute_batch_iterator(connection, iter(beers), page_size=1000)
insert_execute_batch_iterator(page_size=1000)
Time   2.537
Memory 0.2265625

>>> insert_execute_batch_iterator(connection, iter(beers), page_size=10000)
insert_execute_batch_iterator(page_size=10000)
Time   2.585
Memory 25.4453125
```

Мы получили некоторые интересные результаты, давайте разберем их:

* 1: Результаты аналогичны результатам, которые мы получили, вставляя строки одну за другой.
* 100: это размер страницы по умолчанию, поэтому результаты аналогичны нашему предыдущему тесту.
* 1000: Тайминг здесь примерно на 40% быстрее, а памяти мало.
* 10000: Время не намного быстрее, чем при размере страницы 1000, но объем памяти значительно выше.

Результаты показывают, что существует компромисс между памятью и скоростью. В этом случае кажется, что оптимальным вариантом является размер страницы 1000.

### Выполнение с значениями (execute\_values)

Жемчужины в документации psycopg не заканчиваются на execute\_batch. Просматривая документацию, мое внимание привлекла еще одна функция под названием [execute\_values](http://initd.org/psycopg/docs/extras.html#psycopg2.extras.execute\_values):

> Выполните оператор, используя VALUES с последовательностью параметров.

Функция execute\_values генерирует для запроса огромный список VALUES.

Давайте покрутим:

```python
import psycopg2.extras

@profile
def insert_execute_values(connection, beers: Iterator[Dict[str, Any]]) -> None:
    with connection.cursor() as cursor:
        create_staging_table(cursor)
        psycopg2.extras.execute_values(cursor, """
            INSERT INTO staging_beers VALUES %s;
        """, [(
            beer['id'],
            beer['name'],
            beer['tagline'],
            parse_first_brewed(beer['first_brewed']),
            beer['description'],
            beer['image_url'],
            beer['abv'],
            beer['ibu'],
            beer['target_fg'],
            beer['target_og'],
            beer['ebc'],
            beer['srm'],
            beer['ph'],
            beer['attenuation_level'],
            beer['brewers_tips'],
            beer['contributed_by'],
            beer['volume']['value'],
        ) for beer in beers])
```

Импорт пива с помощью функции:

```python
>>> insert_execute_values(connection, beers)
insert_execute_values()
Time   3.666
Memory 4.50390625
```

Таким образом, прямо из коробки мы получаем небольшое ускорение по сравнению с execute\_batch. Однако памяти немного выше.

### Выполнение с значениями из итератора (execute\_values)

Как и раньше, чтобы уменьшить потребление памяти, мы стараемся избегать хранения данных в памяти, используя итератор вместо списка:

```python
@profile
def insert_execute_values_iterator(connection, beers: Iterator[Dict[str, Any]]) -> None:
    with connection.cursor() as cursor:
        create_staging_table(cursor)
        psycopg2.extras.execute_values(cursor, """
            INSERT INTO staging_beers VALUES %s;
        """, ((
            beer['id'],
            beer['name'],
            beer['tagline'],
            parse_first_brewed(beer['first_brewed']),
            beer['description'],
            beer['image_url'],
            beer['abv'],
            beer['ibu'],
            beer['target_fg'],
            beer['target_og'],
            beer['ebc'],
            beer['srm'],
            beer['ph'],
            beer['attenuation_level'],
            beer['brewers_tips'],
            beer['contributed_by'],
            beer['volume']['value'],
        ) for beer in beers))
```

Выполнение функции дало следующие результаты:

```python
>>> insert_execute_values_iterator(connection, beers)
insert_execute_values_iterator()
Time   3.677
Memory 0.0
```

Итак, время почти такое же, но память вернулась к нулю.

### Выполнение с значениями из итератора с размером страницы (execute\_values)

Как и execute\_batch, функция execute\_values также принимает аргумент page\_size:

```python
@profile
def insert_execute_values_iterator(
    connection,
    beers: Iterator[Dict[str, Any]],
    page_size: int = 100,
) -> None:
    with connection.cursor() as cursor:
        create_staging_table(cursor)
        psycopg2.extras.execute_values(cursor, """
            INSERT INTO staging_beers VALUES %s;
        """, ((
            beer['id'],
            beer['name'],
            beer['tagline'],
            parse_first_brewed(beer['first_brewed']),
            beer['description'],
            beer['image_url'],
            beer['abv'],
            beer['ibu'],
            beer['target_fg'],
            beer['target_og'],
            beer['ebc'],
            beer['srm'],
            beer['ph'],
            beer['attenuation_level'],
            beer['brewers_tips'],
            beer['contributed_by'],
            beer['volume']['value'],
        ) for beer in beers), page_size=page_size)
```

Выполнение с разными размерами страниц:

```python
>>> insert_execute_values_iterator(connection, iter(beers), page_size=1)
insert_execute_values_iterator(page_size=1)
Time   127.4
Memory 0.0

>>> insert_execute_values_iterator(connection, iter(beers), page_size=100)
insert_execute_values_iterator(page_size=100)
Time   3.677
Memory 0.0

>>> insert_execute_values_iterator(connection, iter(beers), page_size=1000)
insert_execute_values_iterator(page_size=1000)
Time   1.468
Memory 0.0

>>> insert_execute_values_iterator(connection, iter(beers), page_size=10000)
insert_execute_values_iterator(page_size=10000)
Time   1.503
Memory 2.25
```

Как и в случае с execute\_batch, мы видим компромисс между памятью и скоростью. Здесь также оптимальный размер страницы составляет около 1000. Однако, используя execute\_values, мы получили результаты примерно на 20% быстрее по сравнению с тем же размером страницы, используя execute\_batch.

### Копировать (copy\_from)

В официальной документации PostgreSQL есть целый раздел, посвященный [заполнению базы данных](https://www.postgresql.org/docs/current/populate.html#POPULATE-COPY-FROM). Согласно документации, лучший способ загрузить данные в базу данных — использовать [команду copy](https://www.postgresql.org/docs/current/sql-copy.html).

Чтобы использовать копирование copy из Python, psycopg предоставляет специальную функцию [copy\_from](http://initd.org/psycopg/docs/cursor.html#cursor.copy\_from). Для команды копирования copy требуется файл CSV. Давайте посмотрим, сможем ли мы преобразовать наши данные в CSV и загрузить их в базу данных с помощью copy\_from:

```python
import io

def clean_csv_value(value: Optional[Any]) -> str:
    if value is None:
        return r'\N'
    return str(value).replace('\n', '\\n')

@profile
def copy_stringio(connection, beers: Iterator[Dict[str, Any]]) -> None:
    with connection.cursor() as cursor:
        create_staging_table(cursor)
        csv_file_like_object = io.StringIO()
        for beer in beers:
            csv_file_like_object.write('|'.join(map(clean_csv_value, (
                beer['id'],
                beer['name'],
                beer['tagline'],
                parse_first_brewed(beer['first_brewed']),
                beer['description'],
                beer['image_url'],
                beer['abv'],
                beer['ibu'],
                beer['target_fg'],
                beer['target_og'],
                beer['ebc'],
                beer['srm'],
                beer['ph'],
                beer['attenuation_level'],
                beer['contributed_by'],
                beer['brewers_tips'],
                beer['volume']['value'],
            ))) + '\n')
        csv_file_like_object.seek(0)
        cursor.copy_from(csv_file_like_object, 'staging_beers', sep='|')
```

Давайте разберемся:

* clean\_csv\_value: преобразует одно значение.
  * **Экранирование новых строк**: некоторые текстовые поля содержат символы новой строки, поэтому мы экранируем `\n -> \\n`.
  * **Пустые значения преобразуются в \N**: строка «\N» — это строка по умолчанию, используемая PostgreSQL для обозначения NULL в COPY (это можно изменить с помощью опции NULL).
* csv\_file\_like\_object: создайте объект, подобный файлу, с помощью [io.StringIO](https://docs.python.org/3.7/library/io.html?#io.StringIO). Объект StringIO содержит строку, которую можно использовать как файл. В нашем случае это CSV-файл.
* csv\_file\_like\_object.write: Преобразовать пиво в строку CSV.
  * **Преобразование данных**: здесь выполняются преобразования first\_brewed и valume.
  * **Выберите разделитель**: некоторые поля в наборе данных содержат произвольный текст с запятыми. Чтобы предотвратить конфликты, мы выбираем `«|»` в качестве разделителя (другой вариант — использовать QUOTE).

Теперь давайте посмотрим, окупилась ли вся эта тяжелая работа:

```python
>>> copy_stringio(connection, beers)
copy_stringio()
Time   0.6274
Memory 99.109375
```

Команда copy — самая быстрая из всех, что мы когда-либо видели! При использовании COPY процесс завершился менее чем за секунду. Однако кажется, что этот метод гораздо более расточителен с точки зрения использования памяти. Функция занимает 99 МБ, что более чем в два раза превышает размер нашего файла JSON на диске.

### Копирование данных из строкового итератора (copy\_from)

Одним из основных недостатков использования копирования с помощью StringIO является то, что весь файл создается в памяти. Что, если вместо создания всего файла в памяти мы создадим файлоподобный объект, который будет действовать как буфер между удаленным источником и командой COPY. Буфер будет использовать JSON через итератор, очищать и преобразовывать данные и выводить чистый CSV.

<figure><img src="../../.gitbook/assets/02-fast-load-data-python-postgresql.png" alt=""><figcaption><p>Копирование данных из строкового итератора (yuml.me)</p></figcaption></figure>

Вдохновленные [этим ответом на stackoverflow](https://stackoverflow.com/a/12604375/2000875), мы создали объект, который использует итератор и предоставляет файловый интерфейс:

```python
from typing import Iterator, Optional
import io

class StringIteratorIO(io.TextIOBase):
    def __init__(self, iter: Iterator[str]):
        self._iter = iter
        self._buff = ''

    def readable(self) -> bool:
        return True

    def _read1(self, n: Optional[int] = None) -> str:
        while not self._buff:
            try:
                self._buff = next(self._iter)
            except StopIteration:
                break
        ret = self._buff[:n]
        self._buff = self._buff[len(ret):]
        return ret

    def read(self, n: Optional[int] = None) -> str:
        line = []
        if n is None or n < 0:
            while True:
                m = self._read1()
                if not m:
                    break
                line.append(m)
        else:
            while n > 0:
                m = self._read1(n)
                if not m:
                    break
                n -= len(m)
                line.append(m)
        return ''.join(line)
```

Чтобы продемонстрировать, как это работает, можно создать объект, подобный файлу CSV, из списка чисел:

```python
>>> gen = (f'{i},{i**2}\n' for i in range(3))
>>> gen
<generator object <genexpr> at 0x7f58bde7f5e8>
>>> f = StringIteratorIO(gen)
>>> print(f.read())
0,0
1,1
2,4
```

Обратите внимание, что мы использовали f как файл. Внутри он извлекал строки из gen только тогда, когда его внутренний буфер строк был пуст.

Функция загрузки с использованием StringIteratorIO выглядит следующим образом:

```python
@profile
def copy_string_iterator(connection, beers: Iterator[Dict[str, Any]]) -> None:
    with connection.cursor() as cursor:
        create_staging_table(cursor)
        beers_string_iterator = StringIteratorIO((
            '|'.join(map(clean_csv_value, (
                beer['id'],
                beer['name'],
                beer['tagline'],
                parse_first_brewed(beer['first_brewed']).isoformat(),
                beer['description'],
                beer['image_url'],
                beer['abv'],
                beer['ibu'],
                beer['target_fg'],
                beer['target_og'],
                beer['ebc'],
                beer['srm'],
                beer['ph'],
                beer['attenuation_level'],
                beer['brewers_tips'],
                beer['contributed_by'],
                beer['volume']['value'],
            ))) + '\n'
            for beer in beers
        ))
        cursor.copy_from(beers_string_iterator, 'staging_beers', sep='|')
```

Основное отличие состоит в том, что CSV-файл beers используется по требованию, и данные не сохраняются в памяти после его использования.

Давайте выполним функцию и посмотрим результаты:

```python
>>> copy_string_iterator(connection, beers)
copy_string_iterator()
Time   0.4596
Memory 0.0
```

Большой! Время мало, и память вернулась к нулю.

### Копирование данных из строкового итератора с размером буфера (copy\_from)

## Сводка результатов

## Краткое содержание
