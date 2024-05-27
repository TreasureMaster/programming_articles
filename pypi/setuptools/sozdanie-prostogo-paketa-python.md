# Создание простого пакета Python

#### Блог Christopher Flynn (flynn.gg) от 27-06-2017

Оригинал статьи [здесь](https://flynn.gg/blog/creating-a-python-package/).

## Создание простого пакета Python

Я зарегистрировался на **GitHub** после получения степени доктора философии в 2016 году. Одним из первых репозиториев, которые я создал, был перенос части моего кода **MATLAB** для генерации реализаций [дробного броуновского движения](https://en.wikipedia.org/wiki/Fractional\_Brownian\_motion) на python. По сути, это был просто `README.md` со ссылкой на источники используемых алгоритмов и единственный файл с именем `fbm.py`.

Несколько месяцев назад я заметил, что [репозиторий](https://github.com/crflynn/fbm) набрал несколько звезд и клонируется пару раз в неделю. Я подумал, что если люди будут заинтересованы в этом коде, я должен попытаться реорганизовать его и создать реальный пакет python, который любой мог бы загрузить и реализовать с помощью **pip**. Вот что я сделал. Пакет называется **fbm** и доступен на [pypi](https://pypi.org/project/fbm/).

Я пройдусь по шагам, которые сделал здесь. После рефакторинга кода и добавления некоторых полезных функций структура каталогов пакета выглядела следующим образом. Единый модуль с небольшим набором юнит-тестов в папке **tests**.

```bash
packagename/
    packagename/
        __init__.py
        packagename.py
    tests/
        packagename_tests.py
    requirements.txt
    README.rst
    .gitignore
```

## setup.py

Первым шагом является создание файла с именем `setup.py` в каталоге верхнего уровня проекта, который импортирует функцию настройки из модуля **setuptools**. Пакет **setuptools** всегда следует использовать в пользу **distutils**.

```python
from setuptools import setup


def readme():
    with open('README.rst') as f:
        return f.read()

setup(name='packagename',
      version='0.1.0',
      description='A simple module',
      long_description=readme(),
      license='MIT',
      author='author name',
      author_email='author@email.com',
      url='github_url',
      packages=['packagename'],
      zip_safe=False,
)
```

Это минимум `setup.py`. Несколько примечаний:

* **name** - должно быть имя пакета, которое еще не занято. Вы можете легко выполнить поиск в [pypi](https://pypi.org/), чтобы узнать, занято ли желаемое имя пакета.
* **version** - следует выбирать на основе руководства \[Semantic Versioning] (Семантическое управление версиями). Это не обязательно, но это хорошая практика.
* **description** - должно быть несколько слов, описывающих пакет.
* **long\_description** - можно более подробное описание. В этом примере вы можете видеть, что мы читаем файл `README.rst`. Обратите внимание, что **pypi** обработает файлы **rst** ([ReStructuredText](https://docutils.sourceforge.io/docs/user/rst/quickstart.html)) и импортирует его на страницу, созданную для пакета.
* **license** - это лицензия, которую вы хотите использовать для своего программного обеспечения. Если вы не уверены, вам следует обратиться к [selectalicense](https://choosealicense.com/).
* **author\_name** - это ваше имя.
* **author\_email** - это ваш емэйл (необходимо указать, если вы укажете свое имя).
* **url** - ссылка на ваш проект. Я использовал URL-адрес репозитория.
* **packages** - вы можете получить список пакетов вручную или использовать функцию [find\_packages](https://setuptools.readthedocs.io/en/latest/setuptools.html#using-find-packages) из **setuptools**.
* **zip\_safe** - смотрите [здесь](https://setuptools.readthedocs.io/en/latest/setuptools.html#setting-the-zip-safe-flag). Если вы не уверены, установите значение `False`.

Здесь вы можете указать ряд [дополнительных аргументов](https://setuptools.readthedocs.io/en/latest/setuptools.html#metadata). Вы должны использовать **install\_requires**, если ваш пакет имеет какие-либо зависимости от других пакетов. Если вы хотите включить **README**, **LICENSE** или другие файлы, отличные от Python, вам также следует использовать аргумент **include\_package\_data** как `True`. Вы также можете использовать аргумент **classifiers**, чтобы указать [категориальные теги](https://pypi.org/pypi?%3Aaction=list\_classifiers) для вашего пакета.

Если **include\_package\_data** имеет значение `True`, вам нужно будет создать файл `MANIFEST.in` на верхнем уровне вашего проекта. Внутри файла вы захотите включить в пакет любые дополнительные релевантные файлы, например:

```bash
include README.rst
include LICENSE.txt
```

Вы также захотите изменить файл `.gitignore`, чтобы игнорировать некоторые файлы, которые будут созданы при сборке пакета. Вот как выглядит мой файл игнорирования. Файл `.env` предназначен для работы в [виртуальной среде](https://python-guide.readthedocs.io/en/latest/dev/virtualenvs/). Файл `.tox` предназначен для [пакета **tox**](https://tox.readthedocs.io/en/latest/), который автоматизирует тестирование на основе виртуального окружения.

```bash
# Скомпилированные модули python.
*.pyc
# Папка с дистрибутивом Setuptools.
/dist/
# Метаданные egg Python, восстановленные из исходных файлов с помощью setuptools.
/*.egg-info
# файл Python virtualenv .env
.env
# tox тестирование
.tox/
# упаковка
MANIFEST
build/
```

Собранный вместе каталог вашего пакета может выглядеть примерно так.

```bash
packagename/
    packagename/
        __init__.py
        packagename.py
    tests/
        packagename_tests.py
    requirements.txt
    README.rst
    LICENSE.txt
    MANIFEST.in
    setup.py
    .gitignore
```

## Создание раздачи

Чтобы создать исходный дистрибутив вашего проекта, выполните команду.

```bash
python setup.py sdist
```

Если ваш проект представляет собой чистый Python, вы можете создать универсальный дистрибутив **wheel**.

```bash
python setup.py bdist_wheel
```

Обе эти команды создадут файлы `.tar.gz` и `.whl` соответственно в папке `dist/`.

## Загрузка в PyPI

Чтобы получить пакет на pypi, вам необходимо зарегистрировать аккаунт. Вы должны создать учетную запись на [pypi](https://pypi.org/account/register/) и [testpypi](https://test.pypi.org/), что позволит вам протестировать свою загрузку, прежде чем размещать настоящую вещь на **pypi**. Вам также следует установить пакет **twine** с помощью команды `pip install twine`.

Затем вы должны создать и открыть файл `~/.pypirc` в текстовом редакторе. Это должно выглядеть так, если ваши учетные данные **pypi** и **testpypi** записаны на:

```bash
[distutils]
index-servers =
    pypi
    testpypi

[pypi]
repository=https://pypi.python.org/pypi
username=pypi_username
password=pypi_password

[testpypi]
repository=https://test.pypi.org/legacy/
username=testpypi_username
password=testpypi_password
```

Вы должны быть предельно осторожны с этим файлом, так как вы храните здесь свои личные учетные данные в виде открытого текста.

Мы будем использовать [testpypi](https://test.pypi.org/) для тестирования загрузки нашего пакета. Сначала нам нужно зарегистрироваться на тестовом сервере. Чтобы зарегистрировать проект на **testpypi**:

```bash
python setup.py register -r https://testpypi.python.org/pypi
```

Чтобы загрузить:

```bash
twine upload dist/packagename-version.tar.gz -r testpypi
twine upload dist/packagename-version-py2.py3-none-any.whl -r testpypi
```

Вы должны увидеть, что страница была создана по адресу **https://testpypi.python.org/pypi/packagename/**. Чтобы проверить, что это сработало:

```bash
pip install -i https://testpypi.python.org/pypi packagename
```

Если все сработало, вы можете запустить эту последнюю команду, чтобы загрузить свой пакет в настоящий **pypi** (обрабатывает регистрацию автоматически), заменив имя пакета и версию на правильные значения:

```bash
twine upload dist/packagename-version.tar.gz
twine upload dist/packagename-version-py2.py3-none-any.whl
```

Все готово, и теперь вы можете увидеть свой пакет на pypi!

## Обновление вашего пакета

Если вы планируете поддерживать свой проект, вам нужно будет загрузить новую версию, когда она будет готова. Не забудьте увеличить номер версии в `setup.py` перед сборкой новых дистрибутивов. Когда новый исходный дистрибутив и файлы **wheel** будут готовы, просто запустите:

```bash
twine upload dist/packagename-newversion.tar.gz
twine upload dist/packagename-newversion-py2.py3-none-any.whl
```

И помните, что всегда полезно сначала использовать **testpypi**.

## Дальнейшее чтение

* [Руководство пользователя упаковки Python](https://packaging.python.org/tutorials/packaging-projects/#packaging-and-distributing-projects)
* [Индексирование пакета Python (pypi)](https://pypi.org/)
* [Тестирование индексирования пакета Python (testpypi)](https://test.pypi.org/)
* [Образец пакета PyPA](https://github.com/pypa/sampleproject)
* &#x20;[ReStructuredText](http://docutils.sourceforge.net/docs/user/rst/quickstart.html)
* &#x20;[Semantic Versioning](http://semver.org/)
* Выбор лицензии  [choosealicense](https://choosealicense.com/)
* [Документация setuptools](https://setuptools.readthedocs.io/en/latest/index.html)
* [Аргументы setup()](https://setuptools.readthedocs.io/en/latest/references/keywords.html)
* [Распространение пакетов и wheel](https://packaging.python.org/tutorials/packaging-projects/#packaging-your-project)
* [Использование testPyPI](https://packaging.python.org/guides/using-testpypi/)
* [twine](https://pypi.org/project/twine/)
