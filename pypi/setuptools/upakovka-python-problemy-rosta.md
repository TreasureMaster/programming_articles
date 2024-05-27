# Упаковка Python - Проблемы роста

Оригинал статьи [Bernat Gabor](https://github.com/gaborbernat): [Python packaging - Growing Pains](https://www.bernat.tech/growing-pain/) от 7-02-2019.

Описывает, где упаковка Python находится сегодня, и куда, как надеется Python, будет двигаться дальше. Исследует проблемы, с которыми мы столкнулись при развертывании PEP-517/518, и уроки, которые мы извлекли.

В двух предыдущих сообщениях я рассмотрел, [какие типы пакетов есть в Python](sostoyanie-upakovki-python.md) и [как работает сборка пакетов](upakovka-python-proshloe-nastoyashee-budushee.md), особенно с введением PEP-517/518. Хотя изменения в первую очередь должны были сделать вещи более надежными, при их внедрении и выпуске мы все же столкнулись с несколькими проблемами. В этом посте будут рассмотрены некоторые из них, которые, надеюсь, послужат извлеченными уроками для всех нас и представят некоторые интересные проблемы, которые необходимо решить в будущем.

Глядя на изменения PEP-517 и PEP-518, можно определить, что серверные части сборки (также известные как **setuptools**, **flit**) имели очень мало общего, только чтобы также раскрыть их функциональность через модуль python. Большая часть тяжелой работы приходится на интерфейс сборки, который теперь должен сгенерировать изолированный python, а затем по-новому вызвать серверные части сборки. Когда мы говорим о интерфейсах сборки в настоящее время, наши варианты в основном - **pip** или **poetry** (и **tox** для разработчиков).

Эти проекты поддерживаются сообществом горсткой активных разработчиков в свободное время. Им не платят за это, и они должны быть осторожны, учитывая бесчисленное множество способов использования этих инструментов. Учитывая это, неудивительно, что после принятия PEP потребовалось почти два года, чтобы выпустить первую реализацию. Планирование, тестирование и внедрение продолжались более года в фоновом режиме.

Несмотря на все приготовления, первый выпуск неизбежно сломал несколько пакетов, в основном там, где люди выполняли некоторые операции, которые застали сопровождающих врасплох. Давайте попробуем разобраться в некоторых из этих примеров и как они были решены.

## PEP-518

PEP представляет [формат файла TOML](https://github.com/toml-lang/toml). Формат, специально созданный для удобного чтения/записи конфигураций. В то время как конфигурация упаковки представлена в разделе **build-system**, другие инструменты могут свободно помещать свою конфигурацию в раздел `tool: name`, если они владеют пространством имен PyPi для имени. Этим сразу стали пользоваться различные инструменты (например, [towncrier](https://pypi.org/project/towncrier/), [black](https://pypi.org/project/black/) и т. д.).

Когда [pip 18.0](https://pip.pypa.io/en/stable/news/#id61) (выпущенный 22 июля 2018 г.) добавил поддержку пакетов PEP-518, использующих `pyproject.toml`, изначально сломался, поскольку PEP-518 требовал, чтобы все пакеты, имеющие `pyproject.toml`, указывали раздел **build-backend**. Но пакеты заранее использовали его только в качестве файла конфигурации для этих других проектов, поскольку они не указали его заранее, когда **pip** сталкивался с этими файлами, он просто вызывал ошибки, жалуясь на недопустимые файлы `pyproject.toml`.

## PEP-517

### Проблема с кешем pip wheel.

Способ установки **pip** в мире PEP-517 - сначала создать **wheel**, а затем извлечь его. Чтобы быть в мире PEP-517, **необходимо** указать ключ **build-backend**, в противном случае все интерфейсы в соответствии со спецификацией должны будут вернуться к использованию команд `setup.py`.

Когда **pip** строит **wheel**, он делает это по умолчанию через систему кеширования. Это механизм ускорения, поэтому, если нескольким виртуальным средам требуется одно и то же **wheel**, мы не перестраиваем его, а используем повторно. Операция по сборке **wheel** PEP-517 также использует это преимущество.

Однако это становится проблемой, когда вы отключаете кеш. Теперь нет целевой папки, в которой можно построить **wheel**. Таким образом, сборка не выполняется, см. [прилагаемую проблему](https://github.com/pypa/pip/issues/6158). Проблема проявилась рано, но в массовом порядке, поскольку большинство систем CI работают с этой включенной опцией, всего через день версия `19.0.1` исправила это.

### pyproject.toml не включается в setuptools

Оказывается, на самом деле над бэкэндом сборки больше работы, чем просто раскрытие их API, как описано в PEP-517. Бэкэнд также должен гарантировать, что `pyproject.toml` прикреплен к собранному пакету с исходным кодом, иначе бэкэнд сборки на пользовательском компьютере не сможет его использовать. [setuptools 1650](https://github.com/pypa/setuptools/pull/1650) исправит это для **setuptools**, в более ранних версиях можно просто включить `pyproject.toml`, указав его внутри `MANIFEST.in`.

### Импорт собранного пакета из setup.py

Еще одна неожиданная проблема возникла при импорте пакета из `setup.py`. Версия пакета по соглашению отображается как в качестве метаданных для пакета (в случае **setuptools** внутри `setup.py` в качестве аргумента **version** для функции **setup**), так и в переменной `__version__` в корне пакета. Можно указать содержимое переменной в обоих местах, но тогда становится сложно синхронизировать его.

В качестве обходного пути многие пакеты начали помещать его в файл `version.py` в корне пакета, а затем импортировать его как `from mypy.version import __version__ as version` как из `setup.py`, так и из корня пакета. Это сработало, потому что когда кто-то вызывает скрипт python, текущий рабочий каталог автоматически присоединяется к `sys.path` (так что вы можете импортировать материалы, отображаемые под ним).

Такое поведение добавления текущего рабочего каталога, хотя никогда не было обязательным, было скорее побочным эффектом, чем вызов сборки через `python setup.py sdist`. Поскольку такое поведение является побочным эффектом (а не гарантией), все проекты, импортируемые из файла `setup.py`, должны явно добавлять папку сценариев в путь **sys** в начале сборки.

Вопрос о том, является ли импорт созданного пакета во время упаковки (когда он еще не построен/распространен) хорошей идеей или нет (хотя группа Python Packaging склоняется к этому не так). Тем не менее, дело в том, что, когда **setuptools** открывал свой интерфейс через `setuptools.build_meta`, он решил не добавлять текущий рабочий каталог в системный путь. PEP никогда не требует от внешнего интерфейса делать это добавление, поскольку большинству серверных программ сборки (декларативных по своей природе) это никогда не понадобится. Таким образом, ответственность за такую функциональность возлагалась на интерфейсную часть. **setuptools** соответственно думает, что если пользователям нужна эта функция, они должны быть явно указаны в файле `setup.py` и добавить вручную соответствующий путь к `sys.path`.

Чтобы упростить базовый код pip, **pip** решил включить в PEP-517 всех людей, имеющих `pyproject.toml`, в бэкэнд **setuptools**. Теперь с этой проблемой начали ломаться даже пакеты, которые не выбрали PEP-517. Чтобы исправить это, в **setuptools** был добавлен новый бэкэнд сборки (`setuptools.buildmeta: __legacy__`), предназначенный для использования во внешних интерфейсах по умолчанию, когда бэкэнд сборки не указан; когда проекты добавляют ключ **build-backend**, им также придется изменить свой `setup.py`, чтобы либо добавить исходный корень в их `sys.path`, либо избежать импорта из исходного корня.

### Самонастраивающиеся серверные части

Была поднята еще одна интересная проблема, имеющая гораздо более узкую базу пользователей, но обнаруживающая интересную проблему. В случае, если мы не хотим использовать **wheel** и предоставляем только через исходный код **source distribution**; как решить проблему предоставления бэкенда для бэкенда? Например, **setuptools** сам упаковывается через **setuptools**. Итак, если бы **setuptools** указали это через PEP-517, интерфейс сборки был бы помещен в бесконечный цикл.

Чтобы установить библиотеку **pugs**, сначала нужно попытаться создать изолированную среду. Для этой среды нужны инструменты настройки, поэтому интерфейс сборки должен будет построить **wheel**, чтобы удовлетворить ее. Сама сборка **wheel** инициирует создание изолированной среды, которая снова имеет зависимость сборки от **setuptools**.

Как разорвать эту петлю? Обязать, чтобы все бэкенды сборки отображались как колеса? Разрешить серверные ВМ, которые могут создавать сами себя? Следует ли разрешить этому бэкэнду самостоятельной сборки брать зависимости? Идет долгое обсуждение различных вариантов, плюсов и минусов, поэтому, если вам интересно, обязательно зайдите на [доску обсуждения Python](https://discuss.python.org/t/pep-517-backend-bootstrapping/789) и выскажите свое мнение.

## Заключение

Упаковать код тяжело. Улучшение системы упаковки без каких-либо поломок, когда пользователи могут писать и запускать произвольный код во время упаковки в свободное время, вероятно, невозможно. С PEP-518, однако, теперь зависимости построения явны, а среды построения легко создавать. С PEP-517 мы можем начать переход к более декларативному пространству имен пакетов, которое позволяет пользователям меньше места делать ошибки и предоставлять более качественные сообщения, когда что-то неизбежно пойдет не так.

Допустим, что по мере внесения этих изменений некоторые пакеты могут сломаться, и мы можем запретить некоторые методы, которые работали до сих пор. Мы (специалисты по сопровождению PyPa) не делаем этого недобросовестно, поэтому, когда все же возникают ошибки, пожалуйста, заполните подробный отчет об ошибках, указав, что пошло не так, как вы пытались его использовать и каков ваш вариант использования.

Мы пытаемся по-настоящему улучшить экосистему упаковки, поэтому мы создали репозиторий интеграционных тестов [integration-test](https://github.com/pypa/integration-test), чтобы гарантировать, что в будущем мы сможем отловить хотя бы некоторые из этих крайних случаев, прежде чем они попадут на вашу машину. Если у вас есть какие-либо предложения или требования к какой-либо части упаковки, не стесняйтесь начать обсуждение в разделе «[Обсудить упаковку на форумах Python](https://discuss.python.org/c/packaging/14)» или открыть вопрос для соответствующего инструмента.

На этом пока все, спасибо, что прочитали все это! Я хотел бы поблагодарить [Пола Гэнссла](https://twitter.com/pganssle) за просмотр публикации о серии упаковок и Tech At Bloomberg за то, что позволили мне делать вклады с открытым исходным кодом в мои рабочие часы.