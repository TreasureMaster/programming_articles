# Курс упаковки Python

Оригинал статьи Martin Thoma [Python Packaging Course](https://martin-thoma.com/python-packaging-course/) от 8-02-2020.

Среда Python устарела. Разработка Python началась до появления Интернета. Естественно, такая выращенная среда представляет собой месиво:

![](../../.gitbook/assets/python\_environment.png)

В этом курсе вы узнаете подробности о упаковке Python и о том, как связаны все инструменты.

## Типы приложений

* Библиотека - должна быть включена в другой код. Она никогда не будет выполнена напрямую. У нее НЕТ точки входа.
* Приложение CLI - приложение командной строки должно выполняться в его собственной среде. Оно получает ввод через интерфейс командной строки, может обращаться к локальному хранилищу или делать веб-запросы. Оно может читать переменные среды. Его надо запустить, выполнить задачи и закончить. У него есть точка входа.
* Приложение GUI - подобно приложениям командной строки, но с графическим интерфейсом вместо интерфейса командной строки.
* Служба - подобно приложению командной строки, но никогда не завершается. Она просто работает в фоновом режиме.
* Записные книжки - специальные скрипты, которые в основном используются для анализа данных.

## Среда разработки

Мне нравится оболочка ZSH с плагином [Oh My ZSH](https://github.com/ohmyzsh/ohmyzsh) и [Sublime Text](https://www.sublimetext.com/) в качестве редактора с множеством разных плагинов; я записал некоторые из [моих плагинов](https://martin-thoma.com/sublime-text/) Sublime Text.

Распространенной альтернативой ZSH является [Fish](https://fishshell.com/). Распространенными альтернативами **Sublime Text** являются [Atom](https://atom.io/) и [VS Code](https://code.visualstudio.com/). Если вы хотите большего, [PyCharm](https://www.jetbrains.com/de-de/pycharm/).

Убедитесь, что у вас установлены [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git), [pipenv](https://pypi.org/project/pipenv/), [cookiecutter](https://cookiecutter.readthedocs.io/en/1.7.0/installation.html#install-cookiecutter) и [pre-commit](https://pre-commit.com/).

## Запуск проекта

Предположим, вы хотите разработать новый **awesome\_project**. Затем вы создаете папку и делаете ее репозиторием git:

```bash
$ cookiecutter https://github.com/MartinThoma/cookiecutter-python-package
full_name [Martin Thoma]:
email [info@martin-thoma.de]:
github_username [MartinThoma]:
project_name [Awesome Project]:
project_slug [awesome_project]:
project_short_description [Awesome Project lets you feel the pure awesomeness of awesome.]:
version [0.1.0]:
Select open_source_license:
1 - MIT license
2 - BSD license
3 - Not open source
Choose from 1, 2, 3 (1, 2, 3) [1]:

$ cd awesome_project
$ git init
```

Чтобы гарантировать, что отказ жесткого диска действительно приведет к минимальным потерям в работе, и для обеспечения совместной работы, мы добавляем удаленный репозитарий. [Github](https://github.com/) и [Gitlab](https://about.gitlab.com/) - отличный выбор. После того, как вы создали там пустой репозиторий, добавьте его как удаленный локально:

```bash
$ git remote add origin git@github.com:MartinThoma/awesome_project.git
```

Перед началом разработки рекомендуется убедиться, что у вас не возникнут проблемы из-за разных версий Python / пакетов. **pipenv** - мой любимый инструмент. На данный момент использование Python 3.8 является хорошей идеей ([текущий статус поддержки версий Python](https://devguide.python.org/#status-of-python-branches)).

```bash
# Инициализировать виртуальную среду
$ pipenv --python 3.8

# Импортируйте требования
$ pipenv install -r requirements.txt

# Импортируйте требования разработчика
$ pipenv install --dev --pre -r requirements-dev.txt

# Убедитесь, что репозиторий остается таким красивым
$ pre-commit install
```

### Форматирование

С помощью упомянутого шаблона **cookiecutter** форматирование в значительной степени уже обработано:

* [black](https://github.com/psf/black) - Самоуверенный форматтер, который уважает **PEP8** и реализует много [Flake8](https://flake8.pycqa.org/en/latest/)
* [isort](https://github.com/PyCQA/isort) - Сортирует импорт

Единственное, чего не хватает, - это форматировщик стиля строки документации. Мне очень нравится [формат строки документации numpydoc](https://numpydoc.readthedocs.io/en/latest/format.html).

Если вы хотите узнать больше о форматировании, я рекомендую прочитать мое [руководство по стилю Python](https://martin-thoma.com/python-style-guide/).

### Модульное тестирование

Прежде чем приступить к тестированию приложения, вы должны решить, какую версию Python вы хотите поддерживать. Ориентация должна заключаться в том, какие версии CPython в настоящее время получают обновления безопасности (см. [вопрос SO](https://stackoverflow.com/questions/60126561/what-are-the-currently-supported-python-versions)).

У Python есть несколько фреймворков для тестирования. Используйте [pytest](https://docs.pytest.org/en/latest/). Он чрезвычайно распространен, стабилен и очень прост в использовании.

Шаблон **cookiecutter** устанавливает несколько полезных плагинов:

* **pytest-cov** - Создает отчет о тестовом покрытии. Это поможет вам определить разделы, в которых ошибки не могут быть обнаружены модульным тестом.
* **pytest-black** - Проверяет, был ли нанесен черный цвет.
* **pytest-flake8** - Еще один тест на форматирование.
* **pytest-mccabe** - Убедитесь, что часть вашего кода не слишком сложна.

Вы также можете попробовать **mutmut**. Это может помочь вам определить, какие строки были проверены, но, возможно, недостаточно тщательно.

Чтобы запустить все ваши тесты, выполните **tox**.

### Документация

Если вы разрабатываете библиотеку, вам нужна документация. Для других типов приложений не так много.

Лучшими инструментами являются [Sphinx](https://www.sphinx-doc.org/en/master/) как генератор документации и [readthedocs.org](https://readthedocs.org/) как хостинговые платформы.

### Безопасность

Если вы разрабатываете приложение, вам необходимо обязательно обновить свои зависимости в случае возникновения уязвимостей. Пакет **bandit** и сервисы [snyk.io](https://snyk.io/) / [pyup.io](https://pyup.io/) могут помочь вам обнаружить такие случаи.

### Версии

Хорошая практика - сделать доступным `[module].__version__`. Конечно, она должна быть такой же, как и версия, которую вы видите через `pip freeze`. И тогда было бы неплохо, если бы `git commit` имел [тег git](https://git-scm.com/book/en/v2/Git-Basics-Tagging).

Конечно, все это можно сделать вручную. Если вам нужен инструмент, [bumpversion](https://pypi.org/project/bumpversion/) довольно широко распространен. Однако это не поддерживается. Поэтому некоторые люди используют [bump2version](https://pypi.org/project/bump2version/). Я не уверен, что буду использовать это.

### Структура проекта

```bash
awesome-project  # (git root)
├── awesome_project  # Это пакет
│   ├── cli.py
│   ├── __init__.py  # Требуется до Python 3.3; Я бы все равно добавил это
│   └── _version.py
├── README.md
├── requirements-dev.txt
├── requirements.txt
├── setup.cfg
├── setup.py
├── tests
│   └── test_awesome_project.py
└── tox.ini
```

## Файлы setuptools

Если у вас есть описанная структура проекта, то упаковка не имеет большого значения.

### setup.py

Убедитесь, что на нем есть как минимум

```python
from setuptools import setup

setup()
```

### setup.cfg

`setup.cfg` читается `setuptools.setup ()`. Он может содержать много чего, но минимальный будет выглядеть так:

```bash
[metadata]
name = awesome_project

author = Martin Thoma
author_email = info@martin-thoma.de

# поддерживайте синхронизацию с awesome_project/_version.py
version = 0.1.0

description = Awesome Project lets you feel the pure awesomeness of awesome.
long_description = file: README.md
long_description_content_type = text/markdown

license = MIT license

[options]
packages = find:
python_requires = >= 3.0
```

## Создание раздачи

Создайте исходный файл дистрибутива:

```bash
$ python setup.py sdist
```

Создайте раздачу **wheel**:

```bash
$ python setup.py bdist_wheel
```

## Публикация пакета

После создания загрузите его в PyPI с помощью [twine](https://pypi.org/project/twine/).

Для этого сначала настройте файл `~/.pypirc`:

```bash
[distutils]
index-servers =
  pypi
  pypitest

[pypi]
username: YourUsername
password: plaintext whatever you had

[pypitest]
repository: https://test.pypi.org/legacy/
username: YourUsername
password: plaintext whatever you had

```

Теперь вы можете загрузить созданные ранее дистрибутивы:

```bash
$ twine upload --repository pypitest dist/*

# В качестве альтернативы, если вы хотите подписать его с помощью GPG:
$ twine upload --repository pypitest -s dist/*
```

{% hint style="info" %}
Также существует **python setup.py upload**. Некоторое время он не использовал **https**. Хотя это изменилось, стандартом де-факто по-прежнему является **twine**. По этим [причинам](https://pypi.org/project/twine/).
{% endhint %}

## Модули управления пакетами

### distutils

Не рекомендуется. Используйте **setuptools**.

### distribute

Это был форк **setuptools**, который снова объединили. Используйте **setuptools**.

### setuptools

**setuptools** используется в `setup.py` и дает вам множество команд:

```bash
$ python setup.py --help-commands
Standard commands:
  build             построить все необходимое для установки
  build_py          "build" чистые модули Python (скопировать в каталог сборки)
  build_ext         "build" расширения C/C++ (скомпилировать/связать с каталогом сборки)
  build_clib        "build" библиотеки C/C++, используемые расширениями Python
  build_scripts     "build" скрипты (копирует и исправляет #! строку)
  clean             очистить временные файлы с помощью команды "build"
  install           установить все из каталога сборки "build"
  install_lib       установить все модули Python (расширения и чистый Python)
  install_headers   установить файлы заголовков C/C++
  install_scripts   установить скрипты (Python или другой)
  install_data      установить файлы данных
  sdist             создать исходный дистрибутив (tarball, zip-файл и т. д.)
  register          регистрирует дистрибутив в PyPI
  bdist             создать встроенный (бинарный) дистрибутив
  bdist_dumb        создать встроенный дистрибутив "dumb"
  bdist_rpm         создать дистрибутив RPM
  bdist_wininst     создать исполняемый установщик для MS Windows
  check             выполнить некоторые проверки пакета
  upload            загрузить бинарный пакет в PyPI

Extra commands:
  bdist_wheel       создать дистрибутив "wheel"
  alias             определить синоним для вызова одной или нескольких команд
  bdist_egg         создать дистрибутив "egg"
  develop           установить пакет в 'development mode'
  dist_info         создать каталог .dist-info
  easy_install      найти/получить/установить пакеты Python
  egg_info          создать каталог дистрибутива .egg-info
  install_egg_info  установить каталог .egg-info для пакета
  rotate            удалить старые дистрибутивы, сохранив N новейших файлов
  saveopts          сохраняет предоставленные параметры в setup.cfg или другом файле конфигурации
  setopt            устанавливает параметр в setup.cfg или другом файле конфигурации
  test              запускает модульные тесты после сборки на месте (устарело)
  upload_docs       Загружает документацию в PyPI
  flake8            Запускает Flake8 на модулях, зарегистрированных в setup.py
  
usage: setup.py [global_opts] cmd1 [cmd1_opts] [cmd2 [cmd2_opts] ...]
   or: setup.py --help [cmd1 cmd2 ...]
   or: setup.py --help-commands
   or: setup.py cmd --help
```

## Форматы распространения пакетов

Есть 3 распространенных формата:

* Source distributions
* Egg
* Wheel

**Egg** устарел и может быть заменен либо исходным кодом, либо **wheel** ([источник](https://packaging.python.org/discussions/wheel-vs-egg/)).

## Менеджер пакетов

PIP - это сокращение от «PIP устанавливает Python». Используй это. Не используйте **easy\_install**.

Команды PIP

```bash
  install       Установить пакеты.
  download      Скачайте пакеты.
  uninstall     Удалите пакеты.
  freeze        Вывести установленные пакеты в формате требований.
  list          Список установленных пакетов.
  show          Показать информацию об установленных пакетах.
  check         Убедитесь, что установленные пакеты имеют совместимые зависимости.
  config        Управляйте локальной и глобальной конфигурацией.
  search        Найдите в PyPI пакеты.
  wheel         Создавайте wheel в соответствии с вашими требованиями.
  hash          Вычислить хэши архивов пакетов.
  completion    Вспомогательная команда, используемая для завершения команды.
  debug         Показать информацию, полезную для отладки.
  help          Показать справку по командам.
```

## Смотрите также

* Alexander VanTol: [Pipenv: Руководство по новому инструменту упаковки Python](https://realpython.com/pipenv-guide/)
* packaging.python.org:
  * [Учебник: упаковка проектов Python](https://packaging.python.org/tutorials/packaging-projects/)
  * [Глоссарий](https://packaging.python.org/glossary/)
* Knewton: [Девять кругов ада зависимостей Python](https://medium.com/knerd/the-nine-circles-of-python-dependency-hell-481d53e3e025), 2015
* Martin Thoma: [Как Python/pip обрабатывает конфликтующие транзитивные зависимости ?](https://stackoverflow.com/questions/60084441/how-does-python-pip-handle-conflicting-transitive-dependencies), 2020
* [Сборка и распространение пакетов с помощью setuptools](https://setuptools.readthedocs.io/en/latest/setuptools.html)
* [Tl; DR Legal](https://tldrlegal.com/): сравнение лицензий
* Conda
  * [Сборка пакетов conda с нуля](https://docs.conda.io/projects/conda-build/en/latest/user-guide/tutorials/build-pkgs.html)
  * [Сборка пакетов conda с помощью скелета conda](https://conda.io/projects/conda-build/en/latest/user-guide/tutorials/build-pkgs-skeleton.html#overview)
  * Martin Thoma: [Есть ли смысл создавать пакет conda из пакета PyPI ?](https://stackoverflow.com/questions/59040271/is-there-a-point-in-creating-a-conda-package-from-an-pypi-package), 2019.

Прочие материалы по упаковке:

* Как создать исполняемый файл Windows (.exe) из скрипта Python: для этого вам необходимо работать в системе Windows.
* [py2exe - учебник](http://www.py2exe.org/index.cgi/Tutorial)
* [py2exe - генерация одиночного исполняемого файла](https://stackoverflow.com/questions/112698/py2exe-generate-single-executable-file/113014#113014)
* [компиляция .py в исполняемые файлы Windows и Mac на Ubuntu](https://stackoverflow.com/questions/17709813/compiling-py-into-windows-and-mac-executables-on-ubuntu)
* [Windows .exe\* можно вырезать из Python, разрабатываемого в Ubuntu](https://milkator.wordpress.com/2014/07/19/windows-executable-from-python-developing-in-ubuntu/)
* [Кросс-компиляция скрипта Python в Linux в исполняемый файл Windows](https://stackoverflow.com/questions/2950971/packaging-a-python-script-on-linux-into-a-windows-executable)
