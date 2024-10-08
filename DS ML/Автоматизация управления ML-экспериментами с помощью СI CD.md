---
tags:
  - data
data_type:
  - MLOps
link: https://habr.com/ru/companies/wunderfund/articles/836908/
company: WunderFund
source: habr
author: WunderFund
---
### TL;DR

- Использование рабочих процессов CI/CD (Continuous Integration/Continuous Deployment (Delivery), непрерывная интеграция/непрерывное развёртывание (непрерывная доставка)) для проведения ML‑экспериментов (Machine Learning, машинное обучение) обеспечивает их воспроизводимость, так как вся необходимая информация должна храниться в системе контроля версий.
    
- CI/CD‑решение GitHub (GitHub Actions) пользуется популярностью из‑за того, что оно интегрировано в платформу GitHub, и из‑за того, что им легко пользоваться. GitHub Actions и Neptune — это идеальная комбинация инструментов для автоматизации обучения ML‑моделей и организации экспериментов с ними.
    
- Чтобы приступить к применению CI/CD для управления экспериментами, нужно всего лишь внести незначительные изменения в код обучения модели, обеспечив возможность его автономного запуска на удалённой машине.
    
- Вычислительные ресурсы, которые предлагает сама платформа GitHub Actions, не подходят для крупномасштабных ML‑нагрузок. Но можно воспользоваться собственными ресурсами для хостинга рабочих процессов GitHub Actions.
    

ML‑эксперименты, по своей природе, полны неопределённости и сюрпризов. Небольшие изменения могут вести к огромным улучшениям, но иногда даже самые хитрые уловки не дают результатов.

В любом случае — успешная работа в сфере машинного обучения держится на систематическом применении итеративного подхода к экспериментам и на исследовании моделей. Именно здесь ML‑специалисты часто сталкиваются с беспорядком. Учитывая то, как много путей они могут избрать, им тяжело бывает удержать в поле зрения то, что они уже попробовали, и то, как это отразилось на эффективности работы моделей. Более того — ML‑эксперименты могут требовать много времени. С ними сопряжён риск пустой траты денег на повторные запуски тех экспериментов, результаты которых уже известны.

С помощью трекера экспериментов, вроде [neptune.ai](https://neptune.ai/blog/best-ml-experiment-tracking-tools), можно скрупулёзно логировать сведения об экспериментах и сравнивать результаты разных попыток. Это позволяет выяснять то, какие настройки гиперпараметров и наборы данных вносят положительный вклад в эффективность работы моделей.

Но запись метаданных — это лишь половина секрета успешного ML‑моделирования. Нужно ещё иметь возможность проведения экспериментов таким образом, который позволяет быстро получать нужные результаты. Многие команды дата‑сайентистов, в основе рабочих процессов которых лежит система Git, сочли CI/CD‑платформы идеальным решением.

В этой статье мы исследуем вышеописанный подход к управления ML‑экспериментами и поговорим о том, в каких ситуациях его применение оправдано. Мы уделим основное внимание платформе GitHub Actions — системе, интегрированной в GitHub. Но освещённые здесь идеи применимы и к другим CI/CD‑фреймворкам.

### Почему стоит внедрять CI/CD для проведения экспериментов в сфере машинного обучения?

Эксперимент в сфере ML обычно предусматривает обучение модели и оценку эффективности её работы. Сначала задают конфигурацию модели и алгоритм обучения. Затем запускают обучение на чётко определённом наборе данных. И наконец — оценивают эффективность работы модели на тестовом наборе данных.

Многие дата‑сайентисты любят работать в блокнотах Jupyter. Этот подход хорошо себя оправдывает на [«разведочной» стадии](https://neptune.ai/blog/using-exploratory-notebooks) проекта. Но его применение быстро приводит к тому, что сложно становится следить за тем, какие именно варианты конфигурации модели уже были испытаны.

Даже когда всю необходимую информацию логируют с помощью трекера экспериментов, когда хранят [снепшоты](https://docs.neptune.ai/logging/source_code/) блокнотов и кода, возврат к одной из предыдущих конфигураций часто превращается в довольно‑таки утомительную задачу.

Применяя [систему контроля версий](https://git-scm.com/book/en/v2/Getting-Started-About-Version-Control), наподобие Git, мы легко можем хранить код, пребывающий в определённых состояниях. Мы можем возвращаться к нужному коду, можем создавать на его основе ветви нового кода, каждая из которых идёт своим путём. Ещё можно сравнить разные версии того, что задаёт особенности обучения модели, выявляя отличия одних конфигураций от других.

Правда, тут имеется несколько проблем:

- **Эксперименты можно воспроизвести только тогда, когда имеются чётко определённые характеристики окружения, набора данных и зависимостей.** Предположим, имеется Git‑репозиторий с кодом модели. Если обучение этой модели отлично выполняется на вашем ноутбуке — это не гарантирует того, что так же всё будет у ваших коллег на их ноутбуках. Не гарантирует это и того, что вы сами сможете с тем же успехом обучить модель через пару месяцев.
    
- **Настройка окружения для обучения модели — это часто достаточно трудоёмкая задача.** Нужно установить необходимые исполняемые файлы и зависимости, нужно настроить доступ к наборам данных, вписать куда‑то логин и пароль для доступа к трекеру экспериментов. Если обучение модели занимает много времени или требует специализированного аппаратного обеспечения вроде GPU, ML‑специалист иногда будет замечать, что он тратит больше времени на настройку удалённых серверов, чем на решение задач моделирования.
    
- **Легко забыть о том, чтобы после каждого эксперимента коммитить все необходимые файлы в систему контроля версий.** При быстром запуске некоей последовательности экспериментов легко забыть о том, чтобы добавить в репозиторий код в том виде, в котором он оказывается перед запуском каждого из этих экспериментов.
    

Но не так всё плохо: все эти проблемы можно решить, проводя ML‑эксперименты с использованием CI/CD‑подхода. Благодаря этому, вместо того, чтобы рассматривать выполнение эксперимента и добавление кода в репозиторий как отдельные дела, организуют прямую связь между ними.

Вот как это выглядит:

1. Настраивают эксперимент и коммитят код в Git‑репозиторий.
    
2. Отправляют изменения в удалённый репозиторий (в нашем случае это GitHub).
    
3. Дальше — идут по одному из двух путей, обычно используемых в ML‑командах:
    
    - CI/CD‑система (GitHub Actions в нашем случае) обнаруживает отправку нового коммита и запускает обучение модели, основываясь на коде.
        
    - Рабочий процесс CI/CD запускают вручную, применяя самый свежий код в репозитории, передавая системе модель и параметры обучения в виде входных значений.
        

Такой подход работоспособен только в тех случаях, когда эксперимент полностью определён внутри репозитория, и когда он не предусматривает ручного вмешательства. Поэтому для его применения специалист вынужден включать в код абсолютно всё, что нужно для проведения эксперимента.

![Comparison of a machine-learning experimentation setup without CI/CD and with CI/CD.](https://habrastorage.org/r/w1560/getpro/habr/upload_files/3a3/a79/3f5/3a3a793f53fee6baad0abd1c6158d25a.png "Сравнение конфигураций систем, в которых проводятся эксперименты по машинному обучению. Слева — без использования CI/CD, справа — с CI/CD. Без CI/CD обучение выполняется на локальной машине. Этот подход не гарантирует того, что окружение эксперимента будет где‑то чётко определено, или того, что в удалённом GitHub‑репозитории хранится в точности та версия кода, которая используется в эксперименте. А в той системе, где используется CI/CD, обучение модели производится на сервере, подготовка которого к работе выполнена на основе кода и другой информации, хранящейся в GitHub‑репозитории.")

Сравнение конфигураций систем, в которых проводятся эксперименты по машинному обучению. Слева — без использования CI/CD, справа — с CI/CD. Без CI/CD обучение выполняется на локальной машине. Этот подход не гарантирует того, что окружение эксперимента будет где‑то чётко определено, или того, что в удалённом GitHub‑репозитории хранится в точности та версия кода, которая используется в эксперименте. А в той системе, где используется CI/CD, обучение модели производится на сервере, подготовка которого к работе выполнена на основе кода и другой информации, хранящейся в GitHub‑репозитории.

### Руководство по автоматизации ML-экспериментов с помощью GitHub Actions

Ниже мы пошагово разберём процесс подготовки рабочего процесса GitHub Actions для проведения обучения моделей и для логирования метаданных с помощью Neptune.

Для того чтобы выполнить то, о чём вы прочтёте, вам понадобится GitHub‑аккаунт. Мы исходим из предположения о том, что вы знакомы с Python, с основами машинного обучения, с Git и с GitHub.

Рабочий процесс CI/CD можно добавить к существующему GitHub‑репозиторию, который содержит скрипты обучения модели. Можно и создать новый репозиторий. Если вам любопытно взглянуть на то, как выглядит предлагаемое нами решение — мы [опубликовали](https://github.com/ionicsolutions/neptune-github-actions) полную версию рабочего процесса GitHub Actions и пример обучающего скрипта. Ещё вы можете изучить [этот](https://app.neptune.ai/ionicsolutions/neptune-github-actions/runs) учебный Neptune‑проект.

#### Шаг 1: структурирование обучающего скрипта

Если некто интересуются сферой автоматизации ML‑экспериментов и обучения моделей с помощью CI/CD, то у него уже, наверняка, имеется скрипт для обучения модели на локальной машине (если в вашем случае это не так — в конце этого раздела вы найдёте пример такого скрипта).

Для того чтобы запустить обучение в GitHub Actions, необходимо иметь возможность настроить Python‑окружение и запустить скрипт без ручного вмешательства.

Добиться этого можно, придерживаясь следующих рекомендаций:

- **Создавайте отдельные функции для загрузки данных и для обучения модели.** Это позволяет разделить обучающий скрипт на две части, подходящие для многократного использования. Писать и тестировать эти функции можно независимо друг от друга. Это, кроме того, позволяет, лишь единожды загрузив данные, обучать на них множество моделей.
    
- **Передавайте в скрипт все модели и параметры обучения, которые нужно менять между экспериментами, через командную строку.** Вместо того, чтобы полагаться на смесь стандартных значений, жёстко заданных в коде, переменных окружения и аргументов командной строки, определяйте все параметры с помощью одного метода. Это упростит наблюдение за тем, как именно значения используются в коде, и сделает систему прозрачной для пользователя. Встроенный Python‑модуль [argparse](https://docs.python.org/3/library/argparse.html) даёт всё, что обычно для этого нужно. Но имеются и более продвинутые варианты — вроде [typer](https://typer.tiangolo.com/) и [click](https://typer.tiangolo.com/).
    
- **Повсюду используйте именованные аргументы и передавайте их через словари.** Это защитит вас от ситуации, когда разработчик тонет в десятках параметров, которые обычно нужно задать. Передача словарей позволит залогировать и вывести именно те аргументы, которые использовались при инициализации модели или при запуске сеанса обучения.
    
- **Выводите сведения о том, чем именно занимается скрипт, и о том, какие именно значения он использует.** Возможность видеть выходные данные обучающего скрипта невероятно полезна. Особенно — в тех случаях, когда что‑то идёт не так, как ожидалось.
    
- **Не включайте в код API‑токены, пароли, или ключи доступа.** Даже если репозиторий не является общедоступным, серьёзнейшую угрозу безопасности может представлять наличие всяческих ключей и паролей в системе контроля версий. То же относится и к совместному использованию таких данных разными разработчиками. Вместо этого такие данные нужно передавать в скрипты через переменные окружения во время работы кода. (Если вы не вполне с этим знакомы, но вам нужно загружать обучающие данные из удалённого хранилища или с сервера баз данных, вы можете сразу перейти к шагу 3 или 4 этого руководства, чтобы узнать об одном удобном и безопасном способе обращения с учётными данными).
    
- **Определяйте и фиксируйте зависимости.** GitHub Actions готовит новое Python‑окружение для каждого запуска сеанса обучения, поэтому нужно, чтобы в проекте были бы определены все зависимости. Их версии должны быть зафиксированы. Это позволит получать воспроизводимые результаты. Здесь мы, для решения этой задачи, будем пользоваться файлом `requirements.txt`, но вы вполне можете прибегнуть и к более продвинутым инструментам, вроде [Poetry](https://python-poetry.org/), [Hatch](https://hatch.pypa.io/latest/) или [Conda](https://docs.conda.io/).
    

Вот — полный пример обучающего скрипта для модели [DecisionTreeClassifier](https://scikit-learn.org/stable/modules/generated/sklearn.tree.DecisionTreeClassifier.html#sklearn-tree-decisiontreeclassifier) из `scikit‑learn`. Модель обучается на хорошо известном наборе данных [iris toy](https://scikit-learn.org/stable/modules/generated/sklearn.datasets.load_iris.html#sklearn.datasets.load_iris), который мы будем использовать и в оставшихся разделах этого руководства.

```
from sklearn.datasets import load_irisfrom sklearn.metrics import accuracy_score, f1_scorefrom sklearn.model_selection import train_test_splitfrom sklearn.tree import DecisionTreeClassifierdef load_data():   print("Loading dataset")   iris_dataset = load_iris()   X, y = iris_dataset.data, iris_dataset.target   print(f"Loaded dataset with {len(X)} samples.")   return train_test_split(X, y, test_size=1 / 3)def train(data, criterion, max_depth):   print(“Training a DecisionTreeClassifier”)   print(“Unpacking training and evaluation data…”)   X_train, X_test, y_train, y_test = data   print(“Instantiating model…”)   model_parameters = {       "criterion": criterion,       "splitter": "best",       "max_depth": max_depth,   }   print(model_parameters)   model = DecisionTreeClassifier(**model_parameters)   print("Fitting model...")   model.fit(X_train, y_train)   print("Evaluating model...")   y_pred = model.predict(X_test)   evaluation = {       "f1_score": f1_score(y_test, y_pred, average="macro"),       "accuracy": accuracy_score(y_test, y_pred),   }   print(evaluation)if __name__ == "__main__":   parser = argparse.ArgumentParser()   parser.add_argument("--criterion", type=str)   parser.add_argument("--max-depth", type=int)   args = parser.parse_args()   data = load_data()   train(       data,       criterion=args.criterion,       max_depth=args.max_depth,   )
```

Единственная зависимость этого скрипта — [scikit-learn](https://scikit-learn.org/), поэтому наш файл `requirements.txt` будет таким:

```
scikit-learn==1.4.2
```

Обучающий скрипт можно запустить из терминала такой командой:

```
python train.py --criterion gini --max-depth 10
```

#### Шаг 2: настройка рабочего процесса GitHub Actions

Рабочие процессы GitHub Actions определяют в виде YAML-файлов. Эти файлы нужно размещать в директории GitHub-репозитория `.github/workflows`.

В этой директории мы создадим файл `train.yaml`, содержащий определение рабочего процесса. В этом файле, в начале работы над ним, содержится лишь имя рабочего процесса:

```
name: Train Model
```

Здесь мы воспользуемся триггером [workflow_dispatch](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#workflow_dispatch), который позволяет вручную запускать рабочий процесс из GitHub-репозитория. В блоке [inputs](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#providing-inputs) мы задаём входные параметры, которые мы хотели бы устанавливать при каждом запуске рабочего процесса:

```
on: workflow_dispatch:   inputs:     criterion:       description: "The function to measure the quality of a split:"       default: gini       type: choice       options:         - gini         - entropy         - log_loss     max-depth:       description: "The maximum depth of the tree:"       type: number       default: 5
```

Здесь мы определили входной параметр `criterion` типа `choice`, который можно задать, выбрав одно из трёх возможных значений. Параметр `max-depth` — это произвольное число, которое мы передаём системе (сведения обо всех поддерживаемых типах параметров смотрите в [документации](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_dispatchinputsinput_idtype) по GitHub).

В рабочем процессе содержится одно задание, которое выполняет обучение модели:

```
jobs: train:   runs-on: ubuntu-latest   steps:     - name: Check out repository       uses: actions/checkout@v4     - name: Set up Python       uses: actions/setup-python@v5       with:         python-version: '3.12'         cache: 'pip'     - name: Install dependencies       run: |          pip install --upgrade pip          pip install -r requirements.txt     - name: Train model       run: |         python train.py \           --criterion ${{ github.event.inputs.criterion }} \           --max-depth ${{ github.event.inputs.max-depth }}
```

Этот рабочий процесс загружает код, настраивает Python и устанавливает все зависимости из файла `requirements.txt`. Затем он запускает обучение модели, используя скрипт `train.py`.

После того, как мы закоммитили определение рабочего процесса в репозиторий и отправили в GitHub, новый рабочий процесс появится на вкладке `Actions`. Теперь можно запустить его, воспользовавшись средствами, показанными на следующем скриншоте.

![Manually launching the GitHub Actions workflow from the GitHub UI.](https://habrastorage.org/r/w1560/getpro/habr/upload_files/cb6/b98/4f4/cb6b984f4a616116b13c454121b3a400.png "Ручной запуск рабочего процесса GitHub Actions из пользовательского интерфейса GitHub (иллюстрация подготовлена авторами материала)")

Ручной запуск рабочего процесса GitHub Actions из пользовательского интерфейса GitHub (иллюстрация подготовлена авторами материала)

Для запуска рабочего процесса нужно перейти на вкладку `Actions`, выбрать в боковой панели слева рабочий процесс `Train Model` и щёлкнуть по выпадающему списку `Run workflow` в верхнем правом углу списка запусков рабочих процессов. Затем надо установить входные параметры, и наконец — нажать на кнопку `Run workflow` для запуска рабочего процесса (подробности об этом смотрите в [разделе](https://docs.github.com/en/actions/using-workflows/manually-running-a-workflow) GitHub-документации о ручном запуске рабочих процессов).

Если всё настроено правильно — вы увидите, как в списке появятся сведения о запуске нового рабочего процесса. (Вам может понадобиться обновить страницу в браузере в том случае, если через несколько секунд эти сведения в списке не появятся). Если щёлкнуть по записи о запуске рабочего процесса — можно увидеть то, что скрипт выводит в консоль, и проследить за ходом работы программы по мере того, как средство запуска кода GitHub выполняет рабочий процесс и шаги обучения модели.

#### Шаг 3: добавление в скрипт возможностей по логированию в Neptune

Теперь, когда мы автоматизировали обучение модели, пришло время начать наблюдение за этим процессом с помощью Neptune. Для этого нам понадобится установить дополнительные зависимости и доработать обучающий скрипт.

Для того чтобы клиент Neptune мог бы отправлять собранные данные на сервер, ему нужно знать имя проекта и токен API, который даёт доступ к проекту. Так как мы не хотим хранить секретную информацию, подобную этой, в Git-репозитории, мы передадим её обучающему скрипту через переменные окружения.

Класс [BaseSettings](https://docs.pydantic.dev/latest/api/pydantic_settings/#pydantic_settings.BaseSettings) из библиотеки Pydantic даёт нам удобный способ парсить конфигурационные значения из переменных окружения. Для того чтобы воспользоваться этой библиотекой в нашем Python-окружении — её нужно установить с помощью команды `pip install pydantic-settings`.

В верхней части обучающего скрипта, сразу после блока импортов, добавим класс `settings`, содержащий две записи типа `str`:

```
from pydantic_settings import BaseSettingsclass Settings(BaseSettings):   NEPTUNE_PROJECT: str   NEPTUNE_API_TOKEN: strsettings = Settings()
```

Когда класс инициализируется, он осуществляет чтение переменных окружения с именами, соответствующими именам атрибутов, использованных при его описании. (Тут ещё можно задать для соответствующих переменных значения, применяемые по умолчанию, или воспользоваться любыми другими возможностями моделей [Pydantic](https://docs.pydantic.dev/latest/concepts/models/)).

Далее — мы определяем данные, за которыми мы, при каждом запуске обучения, хотим наблюдать. Для этого, в первую очередь, надо установить клиент Neptune, выполнив команду `pip install neptune`. Если вы прорабатываете это руководство, пользуясь предложенным нами примером скрипта, или если обучаете другую scikit-learn-модель — вам нужно будет наладить [интеграцию](https://docs.neptune.ai/integrations/sklearn/) Neptune и `scikit-learn`, установив соответствующий пакет командой `pip install neptune-sklearn`.

После завершения установки в верхнюю часть текста скрипта `train.py` надо добавить следующие команды импорта:

```
import neptuneimport neptune.integrations.sklearn as npt_utils  # для scikit-learn-моделей
```

Затем, в конец функции `train()`, после кода, отвечающего за обучение и оценку модели, добавим код, инициализирующий новый объект Neptune `run`. Здесь будут использоваться конфигурационные переменные из объекта `settings`, о котором мы говорили выше:

```
run = neptune.init_run(   project=settings.NEPTUNE_PROJECT,   api_token=settings.NEPTUNE_API_TOKEN,)
```

В Neptune объект `Run` — это центральный объект для логирования метаданных эксперимента. Мы можем рассматривать его как словарь, в который можно добавлять данные. Например, в него можно добавлять словари с параметрами модели и с результатами её оценки:

```
run["model/parameters"] = model_parametersrun["evaluation"] = evaluation
```

В run можно добавлять структурированные данные, вроде чисел и строк, а так же — последовательности метрик, изображений и файлов. Для того чтобы узнать о различных возможностях Neptune по логированию данных — обратитесь к [документации](https://docs.neptune.ai/logging/methods/).

В этом примере мы используем средства [интеграции](https://docs.neptune.ai/integrations/sklearn/) Neptune с `scikit-learn`, которые предоставляют вспомогательные функции для различных сценариев использования системы. Например — можно сгенерировать и залогировать матрицу несоответствий и выгрузить обученную модель:

```
run["visuals/confusion_matrix"] = npt_utils.create_confusion_matrix_chart(   model, X_train, X_test, y_train, y_test)run["estimator/pickled-model"] = npt_utils.get_pickled_model(model)
```

Мы завершаем блок, отвечающий за логирование данных, останавливая `run`. Теперь это — последняя строка функции `train()`:

```
run.stop()
```

Полную версию обучающего скрипта можно найти здесь.

Прежде чем вы закоммитите изменения и отправите код в репозиторий — не забудьте добавить `pydantic-settings`, `neptune` и `neptune-sklearn` в свой файл `requirements.txt`.

#### Шаг 4: настройка проекта Neptune и передача идентификационных данных в рабочий процесс

А теперь — последние ингредиенты, которые нам нужны для того чтобы запустить первый эксперимент, над которым будет организовано наблюдение. Это — проект Neptune и соответствующий API‑токен.

Если у вас пока нет аккаунта Neptune — перейдите на [страницу](https://neptune.ai/register) регистрации, где вы можете создать бесплатную персональную учётную запись.

Войдите в своё [рабочее пространство Neptune](https://app.neptune.ai/) и либо [создайте новый проект](https://docs.neptune.ai/setup/creating_project/), либо выберите один из существующих. В левом верхнем углу экрана щёлкните по своему имени пользователя, а затем — по строке `Get your API token`.

![Setting up a Neptune project and pass credentials to the workflow](https://habrastorage.org/r/w1560/getpro/habr/upload_files/96d/452/e39/96d452e39422d74af80fa049d9d823b3.png "Получение API-токена (иллюстрация подготовлена авторами материала)")

Получение API-токена (иллюстрация подготовлена авторами материала)

Скопируйте токен из всплывающего виджета.

Теперь можно вернуться в GitHub-репозиторий и перейти на вкладку `Settings`. Там, в левой боковой панели, нужно выбрать `Environments`, а затем — нужно щёлкнуть по кнопке `New environment` в правом верхнем углу. Окружения (Environments) — это механизм, используемый в GitHub Actions для организации конфигурационных переменных окружения и учётных данных, а так же — для управления доступом к ним

Назовём наше окружение Neptune (вы можете выбрать другое имя, подходящее к конкретному проекту, если вы планируете логировать данные в другие аккаунты Neptune из одного и того же репозитория) и добавим в него секретные данные и переменную.

![Setting up Neptune's project in GitHub repository](https://habrastorage.org/r/w1560/getpro/habr/upload_files/c6f/05d/fbe/c6f05dfbeae326e5594231f4ccbc1ed3.png "Настройка проекта Neptune в GitHub-репозитории (иллюстрация подготовлена авторами материала)")

Настройка проекта Neptune в GitHub-репозитории (иллюстрация подготовлена авторами материала)

Секретные данные, добавленные в поле `NEPTUNE_API_TOKEN` — это API-токен, который мы только что скопировали. А переменная `NEPTUNE_PROJECT` — это полное имя проекта, в которое включено имя рабочего пространства. В то время как значения переменных видны здесь в виде обычного текста, секретные данные хранятся в зашифрованном виде. Доступ к ним можно получить только из рабочих процессов GitHub Actions.

Для того чтобы узнать имя проекта — нужно перейти на страницу обзора проекта в интерфейсе Neptune, найти проект и щёлкнуть по `Edit project information`:

![Finding project name in neptune.ai](https://habrastorage.org/r/w1560/getpro/habr/upload_files/e22/8b2/36f/e228b236f7f4b8023a5495647353f9f1.png "Выяснение имени проекта в Neptune (иллюстрация подготовлена авторами материала)")

Выяснение имени проекта в Neptune (иллюстрация подготовлена авторами материала)

В результате этих действий откроется виджет, где можно изменить и скопировать полное имя проекта.

После того, как мы настроили окружение GitHub — можно настроить рабочий процесс так, чтобы он передавал бы информацию в наш расширенный обучающий скрипт. Для этого нам понадобится внести в систему два изменения:

- В определении задания надо указать имя окружения, из которого нужно получать секретные данные и переменные окружения:
    
    ```
    jobs: train:   runs-on: ubuntu-latest   environment: Neptune   steps:      # …
    ```
    
- В разделе описания шагов обучения надо задать передачу секретных данных и переменной в виде переменных окружения:
    
    ```
    - name: Train model  env:   NEPTUNE_API_TOKEN: ${{ secrets.NEPTUNE_API_TOKEN }}   NEPTUNE_PROJECT: ${{ vars.NEPTUNE_PROJECT }}  run: |   python train.py \     --criterion ${{ github.event.inputs.criterion }} \     --max-depth ${{ github.event.inputs.max-depth }}
    ```
    

#### Шаг 5: запуск обучения и проверка результатов

А теперь, наконец, пришло время увидеть всё это в действии!

Перейдём на вкладку `Actions`, выберем рабочий процесс и запустим его. После того, как обучение завершится, в логе рабочего процесса можно увидеть, как клиент Neptune собирает и выгружает данные.

В интерфейсе Neptune, в окне `Runs`, можно найти записи об эксперименте, собранные с помощью объекта `Run`. Там можно увидеть, что Neptune собрал не только ту информацию, сбор которой был задан в обучающем скрипте, но и некоторые другие сведения, собираемые автоматически:

![https://i0.wp.com/neptune.ai/wp-content/uploads/2024/05/Zrzut-ekranu-2024-05-27-o-13.44.54.png?fit=1020%2C437&ssl=1](https://habrastorage.org/r/w1560/getpro/habr/upload_files/144/9fa/693/1449fa69329f22e629c59b614b858f82.png "Сведения об эксперименте в интерфейсе Neptune")

Сведения об эксперименте в интерфейсе Neptune

Например, тут, в разделе `source_code`, можно найти обучающий скрипт и информацию о Git-коммите, к которому он принадлежит.

Если в скрипте использовалось средство интеграции Neptune с библиотекой `scikit-learn`, и при этом были залогированы полные итоговые сведения о работе кода, то в разделе `summary` (на вкладке `All metadata` или `Images`) можно найти различные диагностические сведения и диаграммы.

![https://i0.wp.com/neptune.ai/wp-content/uploads/2024/05/How-to-automate-ML-experiment-management-with-CICD-6.png?fit=1020%2C538&ssl=1](https://habrastorage.org/r/w1560/getpro/habr/upload_files/1af/2ea/e96/1af2eae96e6189ccefc2cf150a9a24ed.png "Сведения о работе scikit-learn")

Сведения о работе scikit-learn

[Здесь](https://app.neptune.ai/ionicsolutions/neptune-github-actions/runs) можно найти полный пример проекта Neptune.

### Запуск заданий GitHub Actions на собственных серверах

По умолчанию система GitHub Actions выполняет рабочие процессы на серверах, поддерживаемых GitHub. Они называются «[runners](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners/about-github-hosted-runners)» (средства выполнения). Это — виртуальные машины, которые рассчитаны на запуск программных тестов и на компиляцию исходного кода, но не на обработку больших объёмов данных или на обучение больших ML‑моделей.

GitHub, кроме того, предоставляет [возможность](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners) использования для GitHub Actions собственных «средств выполнения». Проще говоря — мы готовим к работе сервер, а GitHub к нему подключается и выполняет на нём задания. Это позволяет сконфигурировать виртуальные машины (или настроить собственное аппаратное обеспечение) с учётом необходимых спецификаций. Например — предусмотреть большие объёмы памяти или наличие GPU.

Для того чтобы настроить собственный сервер, надо перейти на вкладку `Settings`, щёлкнуть по `Actions` в левой боковой панели, а там, в подменю, выбрать `Runners`. В диалоговом окне Runners надо щёлкнуть по кнопке `New self‑hosted runner` в правом верхнем углу.

Благодаря этому будет открыта страница с инструкциями о том, как подготовить машину, которая будет зарегистрирована в качестве «средства выполнения» для GitHub Actions. После того, как собственный сервер готов к работе, нужно лишь изменить параметр `runs‑on` в файле рабочего процесса, записав в него, вместо значения `ubuntu‑latest`, значение `self‑hosted`.

Более подробные сведения об этом, в частности — данные о доступных параметрах и рекомендации по безопасности, ищите в [документации](https://docs.github.com/en/actions/hosting-your-own-runners) по GitHub Actions.

### Итоги

Как видите, это довольно просто — начать применение CI/CD в экспериментах по машинному обучению. Мы, на примере GitHub Actions и Neptune, разобрали весь этот процесс — он написания скрипта, работающего на локальной машине, до полноценного рабочего процесса, организующего машинное обучение, в ходе работы которого выполняется логирование метаданных.

Разработка ML‑системы, основанной на CI/CD, и подгонка этой системы под нужды конкретной команды занимают некоторое время. Здесь нужно разобраться с тем — как именно члены команды используют репозитории и рабочие процессы. Но, сразу же после ввода в строй такой системы, её пользователи получат главное преимущество, которое она даёт: полную воспроизводимость и прозрачность каждого сеанса обучения.

CI/CD‑среда подходит не только для ML‑экспериментов. Здесь, кроме того, можно заниматься [оптимизацией гиперпараметров](https://neptune.ai/blog/how-to-optimize-hyperparameter-search) и [упаковкой моделей](https://neptune.ai/blog/ml-model-serving-best-tools). Некоторые команды, работающие в сфере науки о данных и машинного обучения, организуют всю свою работу вокруг Git‑репозиториев. Эта практика известна как «GitOps».

Но даже если вы всего лишь время от времени обучаете маленькие модели, применение GitHub Actions тоже может принести вам пользу. Это отличный способ обеспечить воспроизводимость результатов обучения моделей и надёжное обновление моделей.