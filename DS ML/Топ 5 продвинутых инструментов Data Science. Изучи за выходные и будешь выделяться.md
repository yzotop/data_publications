---
tags:
  - data
data_type:
  - DS
link: https://habr.com/ru/articles/832856/
source: habr
---
Привет, чемпионы! Давайте сегодня рассмотрим 5 инструментов, которые стоит применять в своих проектах прямо сейчас и становиться круче. Посмотрим, как улучшить ваш код, чтобы он был без запаха, как сделать ваш pipeline более стабильным и фиксировать выбросы, как не писать один код по 10 раз

Сегодня окунемся в работу с:

- Makefile - проект сам соберется на работу
    
- Linters - делает джентельменский код
    
- Lightning - прокачаем PyTorch, забываем про train loop
    
- DVC - правильно храним данные
    
- ClearML - настраивает слежку за обучением
    

### 📦 Makefile - твой код заработает на другой машине, прикинь!

Допустим мы создали свой первый пет-проект. И сейчас наша цель, чтобы этот код запустился и у других, а не только у нас. А чтобы его запустить - надо исполнить целую портянку команд: от установки библиотек, внесения в реестр пути для используемых библиотек, до запуска кода... Согласитесь звучит невкусно

Намного приятнее, когда написал одну команду, а все установки и проверки делаются автоматически. Пользователи Linux могут сказать, что пишем [](http://start.sh/)`start.sh` на bash и все работает. Действительно заработает, но только на Linux и MacOS, также достаточно трудно новичку написать bash-скрипт. Самый простой выход из этой ситуации - **Makefile**

Достаточно создать внутри проекта файлик **Makefile**, со всеми этапами сборки, а пользователь может выбрать действие и запустить необходимый этап. Не приходится задумываться, что происходит под капотом - откинулись на спинку кресла и ждем пока завершится процесс. То что мы хотели!

![Когда узнал о существовании CI/CD](https://habrastorage.org/r/w1560/getpro/habr/upload_files/b20/0a7/326/b200a7326d74e44bd67065de7f29f138.png "Когда узнал о существовании CI/CD")

Когда узнал о существовании CI/CD

Скрытый текст

```

```

То есть вы пишете те же самые команды, но пользователю будет достаточно написать команду `make run_script`в консоле. После чего окружение соберется, и пользователь сможет работать дальше с вашим проектом. Причем все опытные программисты уже знают про эту договоренность и сразу ставят ее без лишних вопросов.

Также правильно написанный **Makefile** вы можете тянуть в любой свой код, изменяя переменные. Так вы сможете сохранить время, которое можно будет потратить на разработку, а не настройку каких-то пакетов.

### 👩‍🏫 Linters - твой код теперь пахнет ромашками

Линтеры - инструменты, которые помогают повышать читабельность вашего кода, а также указывают на ваши ошибки в стиле оформления. Для _Python_: `Black`, `Flake8`, `Pylint`, `Bandit`, `Ruff`. Сегодня заострим внимание на `Pylint`, тк его используют много проектов и он выдает оценку вашего кода. Так что сможете ходить и перед всеми красоваться, как вы написали код на -40 из 10 возможных

![Первый коммит мамкиного программиста](https://habrastorage.org/r/w1560/getpro/habr/upload_files/1c7/420/890/1c74208902749f17f18f88a17b0011ca.png "Первый коммит мамкиного программиста")

Первый коммит мамкиного программиста

Перед нами стоит кейс: собрать все данные о пользователе в удобном для дальнейшей работы формате. За 3 минуты крутейший программист приемный сын маминой подруги выдал вот такой страшный код

Скрытый текст

```

```

Сразу замечаем странное название переменных `dct`, `lst`. Если вы до сих пор так называете свои переменные - перестаньте так их называть! Сегодня мы автоматизируем свою жизнь, так что переходим от ручного рефакторинга к автоматическому с помощью `Pylint`. Пишем команду `pylint file_name.py`

Получаем:

```
main.py:1:0: C0114: Missing module docstring (missing-module-docstring)main.py:1:0: C0103: Constant name "name_per" doesn't conform to UPPER_CASE naming style (invalid-name)main.py:5:0: C0413: Import "import copy" should be placed at the top of the module (wrong-import-position)main.py:9:0: C0200: Consider using enumerate instead of iterating with range and len (consider-using-enumerate)-----------------------------------Your code has been rated at 6.36/10
```

Даже так мы набрали больше 6 баллов (Чтобы набрать -100 надо конкретно попотеть). Данный инструмент вы можете встроить в `git precommit`, чтобы перед коммитом код прогонялся через линтер и если балл ниже определенного, то разработчику надо будет дописать код. Так вы сэкономите кучу времени на дальнейшем рефакторинге кода

### ⚡ Ligthning - подзаряди свой pytorch

**PyTorch Lightning** - целый фреймворк, который следит за обучением, дообучением и выпуском моделей в прод. **Lightning -** это фактически огромная надстройка над **PyTorch**, которая бустит процесс проектирования моделей в разы

Если кратко, как интегрирована Lightning:

![Что умеет PyTorch Lightning](https://habrastorage.org/r/w1560/getpro/habr/upload_files/e5c/2cf/a32/e5c2cfa32ee07db5339d24768608d70b.png "Что умеет PyTorch Lightning")

Что умеет PyTorch Lightning

Давайте представим, что мы реализовали следующую простую модель:

Скрытый текст

```

```

Для обучения нам придется написать следующее:

Скрытый текст

```

```

Так мы действительно обучим модель и получим какие-то показатели. Но если я захочу отслеживать метрики по мере обучения? А как включить мой любимый **Early stopping**? При увеличении модели может понадобится **gradient accumulation**. И все это придется писать вручную

Хорошо, что есть **Lightning**. Мы можем дописать некоторые функции и добавить функционала без лишней головной боли.

Скрытый текст

```

```

Чтобы моделька завелась в Lightning необходимо дописать `training_step`, `test_step`, `validation_step` . Чтобы отслеживать метрики мы дописали `self.log` так мы сможем логировать все что заходим

И опять обучать модельку через циклы? **Нет и еще раз нет!** Создадим "тренера", который следить за обучением вашей модели и показывать все показатели в удобном формате:

```
trainer = pl.Trainer(    accelerator="gpu",     devices=1,     min_epochs=1,     max_epochs=3,     precision=16,)trainer.fit(model, train_loader, val_loader)trainer.validate(model, val_loader)trainer.test(model, test_loader)
```

![Обучение в Lightning](https://habrastorage.org/r/w1560/getpro/habr/upload_files/9e2/64c/01b/9e264c01b4d345ffea6cc741b15e4bb4.png "Обучение в Lightning")

Обучение в Lightning

Если же мы хотим отслеживать какие-то специальные метрики, то:

```
self.log_dict({'train_loss': loss, 'train_accuracy': accuracy, 'train_f1_score': f1_score},                      on_step=False, on_epoch=True, prog_bar=True)
```

Позволит логировать не только лосс, но и **accuracy** и **f1** примерно в таком виде

![Accuracy и F1](https://habrastorage.org/r/w1560/getpro/habr/upload_files/3ba/247/1d6/3ba2471d676b38322d49161e88c9a806.png "Accuracy и F1")

Accuracy и F1

Также можно легко настроить распределенные вычисления, логирование в **MLFlow** и другие сервисы. В общем **Lightning** это огромный фреймворк, который призван убирать рутину в работе и наводить порядок в экспериментах

### 🐈 DVC - свой такой git для датасетов

Самое вкусное оставили на конец. Часто ли у вас бывало, когда хакатоните и у вас собирается 100+ версий исходного датасета, а вы не понимаете на чем обучать модель... Какие фичи были внедрены, какие нужны/не нужны, была ли фильтрация  
**DVC** призван помочь вам в этом деле. Давайте посмотрим как его стоит применять

![Предлагаешь коллегам навести порядок в pipeline](https://habrastorage.org/r/w1560/getpro/habr/upload_files/ecf/aac/0ff/ecfaac0ff78eaf8f4bde36fcc2c9c8b7.png "Предлагаешь коллегам навести порядок в pipeline")

Предлагаешь коллегам навести порядок в pipeline

**DVC (Data Version Control)** умеет ооочень много, но сегодня мы пощупаем эту библиотеку: попробуем создать локальное хранилище, создать некоторые изменения и запушить изменения на сервер.

Создадим новый проект:

```
poetry new dvc_learningcd poetry add dvcdvc init
```

Фактически DVC умеет взаимодействовать со всеми популярными хранилищами данных начиная от Google Drive и до AWS. Но мы будем использовать репу на Github

Любезно воспользуемся [датасетом с kaggle](https://www.kaggle.com/datasets/uciml/red-wine-quality-cortez-et-al-2009) для примера. Фиксируем первоначальные изменения `dvc add data/red_wine.csv`

Появляется еще один файлик - `red_wine.csv.dvc`. В котором записывается служебная информация о нем: хеш файла, по какому пути он располагается и тд. Размер такого файла около 1KB, поэтому не стоит переживать, что у вас будет засорятся репка

Теперь давайте переходить от теории к практике - напишем функцию генерации фичей, а потом прогоним наш условный pipeline

Скрытый текст

```

```

Нагенерировали две фичи и также применили PCA, после этого сохранили все это в файл. Давайте пробовать прогнать наш пайплайн через DVC:

```
dvc run -f feature_generation.dvc \	-d feature_engineering.py \	-d data/winequality-red.csv \	-o data/winequality-featured.csv \	python feature_engineering.py
```

Через аргумент `-f` мы даем понять DVC где он будет сохранять метаданные датасета. `-d` необходимые для сборки проекта. `-o` куда сохраняем новый датасет, и наконец сама команда запуска pipeline. Также вы можете сохранять метрики на вашем новом датасете -  
это решает нашу основную проблему!

Если мы хотим все это дело сохранить на Github, то нам очень повезло тк DVC по умолчанию использует Git для сохранения файлов, поэтому: команда `git init`

После настраиваем Git и в путь: `dvc push`

И вы прекрасны! Изменения появились на сервере. При новых датасетов напишите краткое, но емкое описание новых данных, чтобы коллеги смогли разобрать стоит использовать их при обучении или нет

Также при добавлении датасета, можно прогнать легкую модель и посмотреть на ее производительность. И при `dvc run` можно добавить метрики и таким образом выбрать лучший датасет из генераций

### 📊 ClearML - убийца Weights & Bias?!

**ClearML** - относительно не новый фреймворк, который нацелен на трекинг ваших экспериментов с моделями. Но сейчас он умеет намного больше:

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/635/03c/0de/63503c0de59a9a917f11b69355923265.png)

**ClearML** умеет буквально все, от сохранения артефактов вашего процесса обучения до оркестраций решений. Также он умеет версионировать датасеты, как DVC  
Чтобы изучить все, что этот проект умеет потребуется не одна статья, поэтому давайте посмотрим только на создание pipeline. Наш пробный pipeline: **загрузка данных -> разбиение кросс валидацией -> тест -> получение метрик -> обучение модели и выгрузка.** Давайте начинать!

Скрытый текст

```

```

Все необходимые функции для выполнения кода ☝️ Приступаем к созданию pipeline:

```
pipe = PipelineController(	project='Data Feeling 🏄',	name='ClearML Demo',	version='1.0',	add_pipeline_tags=False)pipe.set_default_execution_queue('default')
```

Обращаемся к классу из **ClearML** - даем название проекта, название таски, версия и будем ли давать теги нашим функция. А дальше добавляем этапы:

Скрытый текст

```

```

Необходимо давать название этапам, также функция которая будет ответственна за степ. Потом передаем все необходимые зависимости для функции - мы их можем брать, как с предыдущего этапа или же сами где-то сохранили. А также если вы хотите поменять выход функции вы это можете сделать в **function_return** и все. Запуск!

![Запускаешь огромный pipeline](https://habrastorage.org/r/w780/getpro/habr/upload_files/049/df0/4ad/049df04ad13bc40ee18d461b43bbdfa6.jpg "Запускаешь огромный pipeline")

Запускаешь огромный pipeline

Для запуска: `pipe.start()` . После чего на странице **ClearML** можем увидеть схему нашего обучения. Можем потыкать и посмотреть на артефакты обучения

Отличительнная черта `ClearML` от других фреймворков в `MLOps` заключается в том, что он старается сохранять все, что можно сохранить. Вы сделали **EDA** в своем проекте - супер, мы их сохраним, чтобы в ничего не потеряли. Обучаете модель через **Lightning**? Мы ее тоже сохраним

Таким образом `ClearML` - отдельная платформа, когда вы хотите настроить логирование обучения без лишней мороки и когда не хотите думать, как сохранять свою модель. А после обучения хотите скинуть своему коллеге графики обучения и метрики, чтобы улучшить модель. Не жизнь, а сказка!

После того, как проект инициализировался мы видим следующую красоту

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/3b2/7bd/d2d/3b27bdd2db71d4155cff15bdf3782e7a.png)

## Выводы

Мы рассмотрели сегодня 5 сочных инструментов, которые увеличат вашу эффективность. Выучите их за ближайшие выходные. Эти инструменты выгодно выделят вас среди остальных коллег.

Делитесь в комментариях, какие инструменты вы используете в работе и почерпнули что-то новое в свой рабочий процесс?  
  
Еще больше про полезные технологии Data Science и ML инструменты вы можете найти в [моем тг-канале](https://t.me/datafeeling). В канале я рассказываю про главные новости в AI и ML, а также пишу про свой практический опыт решения ML кейсов. Например недавно писал про собранную демку [голосовой заказчика](https://t.me/datafeeling/913) в Додо с помощью RAG + few-shots техники.