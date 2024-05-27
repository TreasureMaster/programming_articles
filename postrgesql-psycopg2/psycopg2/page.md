# Page

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

* [Установка: пивоварня](page.md#ustanovka-pivovarnya)
  * [Данные](page.md#dannye)
  * [Получить данные](page.md#poluchit-dannye)
  * [Создать таблицу в базе данных](page.md#sozdat-tablicu-v-baze-dannykh)
* [Метрики](page.md#metriki)
  * [Измерение времени](page.md#izmerenie-vremeni)
  * [Измерение памяти](page.md#izmerenie-pamyati)
  * [Декоратор profile](page.md#dekorator-profile)
* [Проверка](page.md#proverka)
  * [Вставка строк одна за другой (execute)](page.md#vstavka-strok-odna-za-drugoi-execute)
  * [Выполнить много (executemany)](page.md#vypolnit-mnogo-executemany)
  * [Выполнить много из итератора (executemany)](page.md#vypolnit-mnogo-iz-iteratora-executemany)
  * [Выполнить пакетно (execute\_batch)](page.md#vypolnit-paketno-execute\_batch)
  * [Выполнить пакетно из итератора (execute\_batch)](page.md#vypolnit-paketno-iz-iteratora-execute\_batch)
  * [Выполнение пакетно из итератора с размером страницы (execute\_batch)](page.md#vypolnenie-paketno-iz-iteratora-s-razmerom-stranicy-execute\_batch)
  * [Выполнение с значениями (execute\_values)](page.md#vypolnenie-s-znacheniyami-execute\_values)
  * [Выполнение с значениями из итератора (execute\_values)](page.md#vypolnenie-s-znacheniyami-iz-iteratora-execute\_values)
  * [Выполнение с значениями из итератора с размером страницы (execute\_values)](page.md#vypolnenie-s-znacheniyami-iz-iteratora-s-razmerom-stranicy-execute\_values)
  * [Копировать (copy\_from)](page.md#kopirovat-copy\_from)
  * [Копирование данных из строкового итератора (copy\_from)](page.md#kopirovanie-dannykh-iz-strokovogo-iteratora-copy\_from)
  * [Копирование данных из строкового итератора с размером буфера (copy\_from)](page.md#kopirovanie-dannykh-iz-strokovogo-iteratora-s-razmerom-bufera-copy\_from)
* [Сводка результатов](page.md#svodka-rezultatov)
* [Краткое содержание](page.md#kratkoe-soderzhanie)



## Установка: пивоварня

### Данные

### Получить данные

### Создать таблицу в базе данных

## Метрики

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
