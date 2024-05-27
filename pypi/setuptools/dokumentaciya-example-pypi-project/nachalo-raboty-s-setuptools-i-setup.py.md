# Начало работы с setuptools и setup.py

Оригинал статьи Andrew Carter: [Getting Started With setuptools and setup.py](https://pythonhosted.org/an\_example\_pypi\_project/setuptools.html) от 11-2009.

{% hint style="danger" %}
Статья очень сильно устарела, однако некоторые моменты можно использовать для ознакомления.
{% endhint %}

**setuptools** - это богатая и сложная программа. В этом руководстве основное внимание уделяется минимальным основам, необходимым для запуска средств настройки, чтобы вы могли:

* Зарегистрировать свой пакет на pypi.
* Собрать egg, исходный код и "распространяемые файлы" оконной программы установки.
* Загрузить эти «распространяемые файлы» в pypi.

## Установка setuptools и easy\_install

Чтобы установить инструменты настройки, посетите [http://pypi.python.org/pypi/setuptools](http://pypi.python.org/pypi/setuptools) и следуйте инструкциям для своей операционной системы. Кроме того, посетите [http://peak.telecommunity.com/DevCenter/EasyInstall](http://peak.telecommunity.com/DevCenter/EasyInstall), чтобы получить дополнительные инструкции по установке инструментов настройки.

В настоящее время (по состоянию на ноябрь 2009 г.) **setuptools** довольно просто установить для Python версий 2.3–2.6.

## Инструменты настройки начальной загрузки

Если у вас возникли проблемы с настройкой инструментов настройки для вашей платформы, вы можете проверить сценарий «bootstrap» **setuptools** по адресу [http://peak.telecommunity.com/dist/ez\_setup.py](http://peak.telecommunity.com/dist/ez\_setup.py).

Вы можете запустить это так:

```bash
$ python ez_setup.py
```

и он установит **setuptools** для любой версии Python. Например, в Windows:

```bash
$ C:\Python24\python.exe ez_setup.py
```

установит **setuptools** для вашего дистрибутива python24.

## Настройка setup.py

Все функции, которые могут быть включены в файл `setup.py`, выходят за рамки этого простого руководства. Я просто сосредоточусь на очень простом и распространенном формате, необходимом для переноса этого проекта на pypi.

Содержимое `setup.py` - это чистый Python:

```python
import os
from setuptools import setup

# Служебная функция для чтения файла README.
# Используется для long_description. Это приятно, потому что сейчас
# 1) у нас есть файл README верхнего уровня и
# 2) легче ввести файл README, чем вставить необработанную строку ниже ...
def read(fname):
    return open(os.path.join(os.path.dirname(__file__), fname)).read()

setup(
    name = "an_example_pypi_project",
    version = "0.0.4",
    author = "Andrew Carter",
    author_email = "andrewjcarter@gmail.com",
    description = ("An demonstration of how to create, document, and publish "
                                   "to the cheese shop a5 pypi.org."),
    license = "BSD",
    keywords = "example documentation tutorial",
    url = "http://packages.python.org/an_example_pypi_project",
    packages=['an_example_pypi_project', 'tests'],
    long_description=read('README'),
    classifiers=[
        "Development Status :: 3 - Alpha",
        "Topic :: Utilities",
        "License :: OSI Approved :: BSD License",
    ],
)
```

## Структура каталога

Структура каталога пока должна выглядеть так:

```bash
some_root_dir/
|-- README
|-- setup.py
|-- an_example_pypi_project
|   |-- __init__.py
|   |-- useful_1.py
|   |-- useful_2.py
|-- tests
|-- |-- __init__.py
|-- |-- runall.py
|-- |-- test0.py
```

## README

Хорошая идея, украденная с [http://pypi.python.org/pypi/Sphinx-PyPI-upload](http://pypi.python.org/pypi/Sphinx-PyPI-upload), - включить текстовый файл README, в которой описание вашего кода. Это будет видно, когда кто-то, скажем, клонирует ваше репо.

Используя простую функцию `read`, ее легко включить в ключевое слово **long\_description** для функции `setuptools.setup ()`.

## Классификаторы

Действительно хороший веб-сайт - [http://pypi.python.org/pypi?%3Aaction=list\_classifiers](http://pypi.python.org/pypi?%3Aaction=list\_classifiers), на котором перечислены все классификаторы, которые вы можете использовать в вызове установки.

Пример этого веб-сайта:

```bash
Development Status :: 1 - Planning
Development Status :: 2 - Pre-Alpha
Development Status :: 3 - Alpha
Development Status :: 4 - Beta
Development Status :: 5 - Production/Stable
Development Status :: 6 - Mature
Development Status :: 7 - Inactive
Environment :: Console
Environment :: Console :: Curses
Environment :: Console :: Framebuffer
Environment :: Console :: Newt
Environment :: Console :: svgalib
```

## Использование setup.py

Основное использование setup.py:

```bash
$ python setup.py <some_command> <options>
```

Чтобы увидеть все команды, введите:

```bash
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
  upload            загрузить бинарный пакет в PyPI

Extra commands:
  rotate            удалить старые дистрибутивы, сохранив N новейших файлов
  develop           установить пакет в 'development mode'
  saveopts          сохраняет предоставленные параметры в setup.cfg или другом файле конфигурации
  setopt            устанавливает параметр в setup.cfg или другом файле конфигурации
  egg_info          создать каталог дистрибутива .egg-info
  upload_sphinx     Загрузить документацию Sphinx на PyPI
  install_egg_info  установить каталог .egg-info для пакета
  alias             определить синоним для вызова одной или нескольких команд
  easy_install      найти/получить/установить пакеты Python
  bdist_egg         создать дистрибутив "egg"
  test              запускает модульные тесты после сборки на месте (устарело)
  build_sphinx      Собрать документацию Sphinx
  
usage: setup.py [global_opts] cmd1 [cmd1_opts] [cmd2 [cmd2_opts] ...]
   or: setup.py --help [cmd1 cmd2 ...]
   or: setup.py --help-commands
   or: setup.py cmd --help
```

## Intermezzo: файл .pypirc и gpg

Чтобы взаимодействовать с pypi, вам сначала необходимо настроить учетную запись. Перейдите на [http://pypi.python.org/pypi](http://pypi.python.org/pypi) и нажмите **Register**.

Теперь, после регистрации, когда вы запускаете команды `setup.py`, которые взаимодействуют с pypi, вам нужно будет каждый раз вводить свое имя пользователя и пароль.

Чтобы обойти это, поместите файл `.pypirc` в свой каталог **$HOME** в Linux. В Windows вам необходимо установить переменную окружения **HOME**, чтобы она указывала на каталог, в котором находится этот файл.

Структура файла `.pypirc` довольно проста:

```bash
[pypirc]
servers = pypi
[server-login]
username:your_awesome_username
password:your_awesome_password
```

{% hint style="info" %}
Вероятно, в этом файле есть ваш простой текстовый пароль, но я не знаю решения и не изучал его.
{% endhint %}

Кроме того, вы часто хотите подписать файлы, используя шифрование **gpg**. Посетите [http://www.gnupg.org/](http://www.gnupg.org/) в Linux или [http://www.gpg4win.org/](http://www.gpg4win.org/) в Windows, чтобы установить это программное обеспечение.

## Регистрация вашего проекта

С вашими `setup.py` и `.pypirc` зарегистрировать проект довольно просто. Просто введите:

```bash
$ python setup.py register
```

Я бы сказал больше, но это так просто.

## Загрузка вашего проекта

Мы будем использовать три основных команды `setup.py`:

* **bdist\_egg**: создает файл egg. Это то, что необходимо, чтобы кто-нибудь мог использовать `easy_install your_project`.
* **bdist\_wininst**: это создаст `.exe`, который установит ваш проект на машине Windows.
* **sdist**: это создает дистрибутив с исходным кодом, который кто-то может напрямую загрузить и запустить `python setup.py`.

{% hint style="info" %}
Ключевым моментом здесь является то, что вам нужно запускать эти команды с той версией python, которую вы хотите поддерживать. Мы расскажем об этом в разделе «[Собираем все вместе с полным сценарием Windows](nachalo-raboty-s-setuptools-i-setup.py.md#sobiraem-vse-vmeste-s-polnym-skriptom-windows)» ниже.
{% endhint %}

Вы можете запускать эти команды сами по себе и просто создавать файлы, но не загружать их. Однако в этом проекте мы всегда объединяем эти команды с директивой **upload**, которая будет создавать и выгружать необходимые файлы.

## Собираем все вместе с полным скриптом Windows

Этот проект был построен на машине с Windows. Чтобы лучше понять, как все это работает, и другие параметры, используемые при использовании `setup.py`, давайте просто посмотрим на файл `.bat`, который я использую для создания пакета и загрузки его в pypi:

```bash
set HOME=C:\Users\Owner\
cd C:\eclipse\workspace\HG_AN_EXAMPLE_PYPI_PROJECT
C:\Python24\python.exe setup.py bdist_egg upload --identity="Andrew Carter" --sign --quiet
C:\Python25\python.exe setup.py bdist_egg upload --identity="Andrew Carter" --sign --quiet
C:\Python26\python.exe setup.py bdist_egg upload --identity="Andrew Carter" --sign --quiet
C:\Python24\python.exe setup.py bdist_wininst --target-version=2.4 register upload --identity="Andrew Carter" --sign --quiet
C:\Python25\python.exe setup.py bdist_wininst --target-version=2.5 register upload --identity="Andrew Carter" --sign --quiet
C:\Python26\python.exe setup.py bdist_wininst --target-version=2.6 register upload --identity="Andrew Carter" --sign --quiet
C:\Python26\python.exe setup.py sdist upload --identity="Andrew Carter" --sign
pause
```

Для Linux это были бы почти те же команды, просто меняя каталоги, чтобы указывать на правильные версии python.

{% hint style="info" %}
Я использую `set HOME = C:\Users\Owner\` вместо установки переменной окружения в Windows
{% endhint %}
