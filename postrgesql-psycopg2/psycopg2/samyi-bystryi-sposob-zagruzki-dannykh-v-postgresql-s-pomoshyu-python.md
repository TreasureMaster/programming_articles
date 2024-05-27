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

### Измерение памяти

### Декоратор profile

## Проверка

### Вставка строк одна за другой (execute)

### Выполнить много (executemany)

### Выполнить много из итератора (executemany)

### Выполнить пакетно (execute\_batch)

### Выполнить пакетно из итератора (execute\_batch)

### Выполнение пакетно из итератора с размером страницы (execute\_batch)

### Выполнение с значениями (execute\_values)

### Выполнение с значениями из итератора (execute\_values)

### Выполнение с значениями из итератора с размером страницы (execute\_values)

### Копировать (copy\_from)

### Копирование данных из строкового итератора (copy\_from)

### Копирование данных из строкового итератора с размером буфера (copy\_from)

## Сводка результатов

## Краткое содержание
