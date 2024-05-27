# Упаковка проекта Python: простой пример

Оригинал статьи Felipe: [Package a Python Project and Make it Available via pip install: Simple Example](https://queirozf.com/entries/package-a-python-project-and-make-it-available-via-pip-install-simple-example) от 25-12-2020.

Вот пример проекта, созданного в соответствии с инструкциями в этом посте:  [Python Data Science Utils](https://github.com/queirozfcom/python-ds-util)

Для получения дополнительной информации о том, что означают такие термины, как pip, virtualenv и т. д., см.  [Python Environment Cheatsheet](http://queirozf.com/entries/python-environment-cheatsheet)

## Образец файла setup.py

Файл `setup.py` - самый важный файл конфигурации для вашего проекта. Вот пример для начала:

```python
# -*- coding: utf-8 -*-

import setuptools

with open("README.md", "r") as fh:
    long_description = fh.read()

setuptools.setup(
    name="myproject",
    version="0.0.1",                        # Обновляйте это для каждой новой версии
    author="Your name",
    author_email="your@email.com",
    description="long description",
    long_description=long_description,
    long_description_content_type="text/markdown",
    install_requires=[                      # Добавьте сюда зависимости проекта
        "pandas>=0.20.0"                    # пример: pandas версии 0.20 или выше                          
    ],                                             
    url="https://github.com/your/github/project",  
    packages=setuptools.find_packages(),
    classifiers=(                                 # Классификаторы помогают людям находить 
        "Programming Language :: Python :: 3",    # ваши проекты. См. все возможные классификаторы 
        "License :: OSI Approved :: MIT License", # на https://pypi.org/classifiers/
        "Operating System :: OS Independent",   
    ),
)
```

## Структура проекта

```bash
project-name/        <--- имя проекта, не обязательно совпадать с внутренним именем
│             
├── myproject        <--- имя модуля, это имя, которое вы будете использовать в "import"                   
│   ├── __init__.py  <--- файл инициализации должен быть здесь, даже если ваш проект - только Python 3
│   └── module.py    <--- подмодули
│ 
├── LICENSE          <--- необязательно, но рекомендуется  
│ 
├── README.md
│ 
└── setup.py         <--- информация о пакете: имя, автор, версия и ЗАВИСИМОСТИ
```

## Упаковка проекта

Чтобы упаковать проект, запустите следующий код (желательно под **virtualenv**):

```bash
(test-venv)$ python setup.py sdist bdist_wheel
running sdist
running egg_info
creating myproject.egg-info
writing requirements to myproject.egg-info/requires.txt
...
...
```

**После выполнения команд упаковки** ваш проект должен выглядеть так:

```bash
project-name/
│ 
├── build
│   ├── bdist.linux-x86_64
│   └── lib
│       └── myproject
│           ├── __init__.py
│           └── module.py 
├── dist                                     <--- это каталог, который
│   ├── myproject-0.0.1-py3-none-any.whl          будет загружен в pypi
│   └── myproject-0.0.1.tar.gz
│ 
├── LICENSE
│ 
├── myproject
│   ├── __init__.py
│   └── module.py
│ 
├── myproject.egg-info
│   ├── dependency_links.txt
│   ├── PKG-INFO
│   ├── SOURCES.txt
│   └── top_level.txt
│ 
├── README.md
│ 
└── setup.py
```

## Загрузка проекта в PyPi

{% hint style="info" %}
Попытка загрузить проект **myproject** не удастся, потому что кто-то уже создал проект на PyPi с таким именем!
{% endhint %}

Для загрузки проекта вам понадобится инструмент под названием **twine**, который можно установить с помощью **pip**:

```bash
(test-venv)$ pip install twine
Collecting twine
...
...
```

На этом этапе вы должны **зарегистрировать учетную запись** на [pypi.org](https://pypi.org/account/register/), чтобы вы могли загружать пакеты.

После создания учетной записи вы можете загружать упакованные файлы

{% hint style="info" %}
Ожидается сообщение об ошибке, поскольку такое имя уже существует. Вы должны выбрать уникальное название проекта.
{% endhint %}

```bash
(test-venv)$ twine upload dist/*
Uploading distributions to https://upload.pypi.org/legacy/
Enter your username: xxxxx
Enter your password: 
Uploading myproject-0.0.1-py3-none-any.whl
100%|████████████████████████████████████████████████████████| 4.52k/4.52k [00:00<00:00, 5.46kB/s]
HTTPError: 403 Client Error: The user 'xxxxx' isn't allowed to upload to project 'MyProject'. 
See https://pypi.org/help/ # project-name for more information. for url: https://upload.pypi.org/legacy/

```

Поздравляю! После загрузки проекта (как указано выше) любой может установить ваш проект через **pip**.

## Исправление проблем

### Не удается импортировать модуль

Если вы не можете импортировать только что загруженный модуль, убедитесь, что у вас есть файл `__init__.py`, как в инструкциях, даже если ваш проект - это полный Python 3!

### Модуль pkg\_resources не имеет атрибута iter\_entry\_points

Принудительно переустановите **setuptools**:

```bash
(venv)$ pip install --upgrade --force-reinstall setuptools
```

{% hint style="info" %}
Этот короткий пост является частью информационного бюллетеня. Нажмите здесь, чтобы [зарегистрироваться](https://queirozf.com/newsletters/data).
{% endhint %}

## Ссылки

* &#x20;[The Hitchhiker's Guide to Packaging](http://the-hitchhikers-guide-to-packaging.readthedocs.io/en/latest/creation.html)
