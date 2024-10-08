---
tags:
  - data
link: https://habr.com/ru/companies/cinimex/articles/824786/
source: habr
data_type:
  - DevOps
company: Cinimex
author: Cinimex
---
![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/32c/bc7/0b4/32cbc70b44024a32c160dea98f358ade.png)

Привет!  
Коротко о себе:

Мой опыт в разработке ПО насчитывает порядка 18 лет, и 6 из них пришлись на работу в качестве Data Scientist. За это время я прошел путь от научного работника, аналитика данных, дата-сайентиста до Chief Data Scientist в банке. Сейчас я работаю в Синимекс, мы занимаемся разработкой ИТ-систем для бизнеса.

В этой статье я бы хотел обозначить и обратить внимание сообщества на проблемы, а также побудить коллег по Data Science инженерии подключиться к инициативе развития MLOps, чтобы совместными усилиями улучшать IT- ландшафт.

![](https://habrastorage.org/getpro/habr/upload_files/c2d/7fb/a93/c2d7fba939dada736ed6a03ab9b7e4d2.JPG)

Если проводить аналогию, каждый Data Science проект можно представить как нежный росток, из которого требуется вырастить большое цветущее дерево.  
  
Как садовник использует для ухода за растениями различные инструменты, так и MLOps инженер использует ML и MLOps инструменты для того чтобы команда смогла прийти к конечному результату. Всего на рынке существует около 300 различных ML-инструментов для интеграции. Это достаточно серьёзное количество, и серьезный вызов для инженеров по выбору наилучшего технологического стека. Невозможно опробовать каждый, поскольку с каждым из них нужно разобраться и найти свое применение.

![ML-инструменты](https://habrastorage.org/r/w1560/getpro/habr/upload_files/fc9/dfe/438/fc9dfe438299a06c5ec800ccff33d5d6.png "ML-инструменты")

ML-инструменты

Однако для того, чтобы разобраться в вопросе, можно попробовать все доступные инструменты отсортировать или сгруппировать тем или иным образом. Ниже представлен один из способов сортировки, связанный с функциональностью инструментов.

![Сортировка инструментов по функциональности](https://habrastorage.org/r/w1560/getpro/habr/upload_files/098/6c8/43a/0986c843acaddd4b97c93958ee94838a.png "Сортировка инструментов по функциональности")

Сортировка инструментов по функциональности

Другой из возможных вариантов группировки MLOps-инструментов — это сортировка по этапам развития DS-проекта:

![Сортировка по этапам развития DS-проекта](https://habrastorage.org/r/w1560/getpro/habr/upload_files/cac/235/388/cac2353883813d841e7417885b2e5c26.png "Сортировка по этапам развития DS-проекта")

Сортировка по этапам развития DS-проекта

Часто инструменты не удовлетворяют всем требованиям и нуждам DS-  
разработки, которая в настоящее время бурно развивается. Разработчикам приходится настраивать, менять, выбирать определенную область применения и создавать свой собственный стек, который подходит для решения задач конкретной компании.

### Что такое MLOps?

Термин MLOps был озвучен на тематических конференциях и в профессиональных кругах в 2018 году и представляет собой сочетание машинного обучения (ML) и практик непрерывной разработки (DevOps). **MLOps** — это набор практик, нацеленных на надежное и эффективное развертывание и поддержание моделей машинного обучения в промышленной среде.

С 2017 года по 2020 год в сфере машинного обучения наблюдался удвоенный рост как тестовых, так и реализованных, т. е. вышедших в production,  проектов. Однако большинство, около 88% этих проектов не преодолевали этап тестирования и не доводились до стадии production. С другой стороны, организации, которые смогли внедрить такие AI и ML-проекты, смогли увеличить рентабельность на 3-15%. Тут можем сделать вывод, что необходимо внедрять технологии, которые позволят бесшовно и внедрять AI и ML-решения.

В 2019 году перспективы рынка ML-решений оценивались приблизительно в 23 миллиарда долларов, а ожидаемый объем рынка в 2025 году — 126 миллиардов долларов, что делает эту область еще более интересной и перспективной.

### Жизненный цикл ML-проекта 

Жизненный цикл ML-проекта, как правило, включает шесть этапов:

1. Анализ бизнеса
    
2. Исследование
    
3. Обучение модели
    
4. Тестирование прототипа
    
5. Развертывание модели
    
6. Сопровождение
    

![Задачи этапов жизненного цикла ML/DS-проекта](https://habrastorage.org/r/w1560/getpro/habr/upload_files/b04/603/aa9/b04603aa9205173a8ffa8588a5eb7858.png "Задачи этапов жизненного цикла ML/DS-проекта")

Задачи этапов жизненного цикла ML/DS-проекта

На рынке мало инструментов, которые покрывают весь жизненный цикл проекта и всегда приходится что-то достраивать. В качестве примеров таких End2End инструментов можно привести следующие и их краткие характеристики:

1.     Databricks — ускоряет разработку ML через весь жизненный цикл проекта

2.     Microsoft Azure — упрощает и ускоряет жизненный цикл ML

3.     Kubeflow — ускоряет жизненный цикл ML при помощи контейнеризированных DS-инструментов  
  
Рассмотрим каждый из этапов жизненного цикла ML-проекта отдельно.

### Анализ бизнеса

Анализ бизнеса можно сравнить с зернышками, из которых мы стремимся вырастить наше прекрасное цветущее дерево.

![Анализ бизнеса](https://habrastorage.org/r/w1560/getpro/habr/upload_files/d35/8a9/43e/d358a943e7825da42f8b5183d11400e8.png "Анализ бизнеса")

Анализ бизнеса

С точки зрения Data Science и развития проекта на этом этапе важно правильно и четко сформулировать бизнес-гипотезы, чтобы сформировать ТЗ на разработку для дата-сайентистов. Очень важно сравнивать различные бизнес-гипотезы с учетом результатов, получаемых в результате развития DS-проекта. Во избежание хаоса лучше все документировать, например, Confluence и Wiki.

На этом этапе мы задействуем заказчика и исполнителя, т.к. формируем тех метрики и правила, которыми мы будем руководствоваться при обучении нашей модели. Очень часто бизнес предлагает свое видение, которое может меняться с учетом предлагаемых нами инициатив. Это может приводить к появлению дополнительного ТЗ, и такой "пинг-понг" может длиться достаточно долго. При взаимодействии с бизнесом необходимо учитывать несколько важных аспектов:

- Во-первых, компании необходимо обладать компетенциями, чтобы оценивать DS-проект, в том числе чтобы провести первоначальный анализ и определить, решаема ли поставленная задача с точки зрения ML.
    
- Во-вторых, бизнес должен понимать, что на первоначальный анализ необходимо потратить некоторые ресурсы и деньги. Чтобы этот процесс не был хаотичным, необходимо определить бюджет первоначального анализа и учитывать его при определении срока анализа. По нашему опыту, разведочный анализ может занимать около двух недель, после чего необходимо выкатывать модель, отказываться от проекта или инициировать какой-то другой проект. 
    

### Исследование

Этап исследования можно сравнить с этапом проращивания зерен для нашего цветущего дерева.

![Исследование](https://habrastorage.org/r/w1560/getpro/habr/upload_files/d62/f25/e87/d62f25e879d577db3b6dbd408ae1ff59.png "Исследование")

Исследование

- Важно, чтобы дата-сайентисты располагали гибким инструментарием, позволяющим быстро проводить исследование и эксперименты без какой-либо настройки.
    
- Необходимо документировать результаты исследований, чтобы к ним было удобно обращаться, например, сравнивать результаты бизнес-гипотез и делиться ими с коллегами.
    
- Процесс исследования должен быть выстроен. Важно поддержать один из существующих процессов разработки, например [LeanDS](https://ods.ai/projects/leands), который позволяет всё организовать. Для этих целей можно использовать Jira, Miro, Trello и т. д.
    

Помимо моделирования и подходов к моделированию, необходимо поработать с данными. На старте DS-проекта приходится обрабатывать большой объем данных, необходимых для реализации проекта. На рынке уже доступно много платных фреймворков и open-source решений, которые позволяют создавать систему управления данными: ведут реестры источников, обеспечивают визуализацию, контролируют качество данных и позволяют конструировать витрины. При этом мало какие решения поддерживают такую важную функцию, как версионирование выгрузок, которая позволяет понимать, на каких данных мы построили модель, а также связывать с ними различные версии модели. Эти два компонента и есть тот уникальный продукт, который мы впоследствии можем предоставить.

### Обучение модели

**1.** Первый шаг в обучении модели- **разметка данных**. Данные должны быть особым образом обработаны, например, изображениям должны быть присвоены метки, которые обозначают соответствующие предметы на нем. Этот процесс трудоемкий и занимает много времени. Чтобы получить качественную модель и обучить ее, данные необходимо проредить (создать подвыборки). Для этого нам необходимы фреймворки и средства, обеспечивающие беспрепятственное развитие проекта, а именно средства коммуникации с людьми, которые будут проводить разметку данных. Такие специалисты могут быть менее квалифицированы, и их можно привлечь со стороны, однако до них необходимо четко донести, как именно нужно работать с данными.

**Средства для развертывания сервиса разметки** могут включать разные решения, например, решение CVAT, которое может быть развернуто on-premise. За работой развернутого решения необходимо следить и поддерживать его работоспособность, и именно в этот момент и возникают вызовы для DevOps. Можно также воспользоваться облачными сервисами, например, Yandex Toloka или Amazon MTurk. При использовании облачных сервисов важной задачей является контроль расходов.

![Обучение модели](https://habrastorage.org/r/w1560/getpro/habr/upload_files/dc8/f29/61e/dc8f2961e4f02f751633bf5ad0396b48.png "Обучение модели")

Обучение модели

**2.** **Формирование кода** для обучения моделей. На этом этапе важно предоставить дата-сайентистам возможность использовать привычные и удобные инструменты, например Jupyter, VSCode или PyCharm, т. е. среды разработки. Как и в процессе обычной разработки программного обеспечения, необходима возможность версионировать код и, в особенности, версионировать данные, т. е. каждой версии кода должна соответствовать версия данных, поскольку именно это создает дает модель, которую мы обучаем.

**3.** **Масштабирование обучения**. Обучение модели не всегда возможно на одной машине, поэтому обучение необходимо масштабировать. По этой причине возникает потребность в создании локальных или облачных вычислительных кластеров, а также необходимость в решении сопутствующих задач, таких как изоляции ресурсов, масштабировании ресурсов по RAM, CPU, GPU, HDD и окружению ОС.

При переезде с локальных машин на кластер, например OpenStack, дата-сайентист получает рабочую станцию, которая является тонким клиентом, и возможность выхода в облако, где разворачивается рабочая среда с необходимыми ресурсами, которыми не всегда располагают локальные системы. Кроме этого, в облаках решается проблема изоляции экспериментов, и дата-сайентисты располагают достаточным количеством ресурсов и не мешают друг другу.

![Обучение модели](https://habrastorage.org/r/w1560/getpro/habr/upload_files/3fc/6c2/90a/3fc6c290a82b09aefa6c01a22e2b9d8d.png "Обучение модели")

Обучение модели

**4.** **Трекинг экспериментов и активное обучение**. При обучении модели необходимо учитывать много различных дополнительных параметров, например, количество подстраиваемых переменных или переменных управляющих ходом обучения. И различные комбинации значений параметров определяют отдельные эксперименты, в рамках которых запускается обучение, получаются результаты. Таких экспериментов могут быть тысячи. Поэтому необходимо логирование и визуализация результатов экспериментов, чтобы можно было выбрать более удачные эксперименты.

**5.** Дополнительно можно развернуть сервис **активного обучения/дообучения**, который позволяет сократить объемы датасетов и выбрать из них только самые информативные примеры, которые привносят в модель наибольшее количество информации.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/2ce/ea4/a8d/2ceea4a8d6abf9df394ba596260c459c.png)

Обучение модели

### ПРОМ-тестирование

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/1d5/d69/1f6/1d5d691f6941854b72b342f8e67f8995.png)

Как правило, ПРОМ-среда не адаптирована под эксплуатацию каких-либо моделей. Для адаптации среды необходимо:

- настроить потоки данных, которые заходят на модель различными способами:
    
    - пайплайны обработки данных (батч-потоки, шины данных, брокеры сообщений, потоки request-response);
        
    - контроль по времени обработки запросов (real-time или отложенная обработка).
        
- завести петлю обратной связи от окружающей среды в базу данных, т. е. получать результаты предсказаний, с которыми работала модель (например, получение кредита и его погашение в связи с предложением взять кредит).
    
- привязать полученные результаты к используемой версии модели, чтобы иметь возможность оценить ее качество в будущем.
    

### Методы тестирования прототипа для оценки качества дообученной модели

- Чтобы оценить качество новой модели, можно использовать исторические данные. Недостатком такого подхода является так называемый “эффект сдвига данных”, который искажает качество. Поскольку меняется не только среда, а еще люди и их приоритеты, модель, откалиброванная на отложенных данных, может некачественно работать в реальности.
    
- Можно создать систему бэк-тестирования, например, смоделировать среду с изменяемыми параметрами, в которой можно получать разные варианты ответов. Однако такая система бэк-тестирования также не гарантирует того, что реальная среда будет себя вести таким же образом, то есть это не спасает от переобучения модели на тестовой выборке.
    
- Поэтому, лучше всего тестировать модель на «живых» данных (онлайн), т. е. в проде. Именно тестирование на «живых данных» можно считать качественным. При этом вопросы защиты персональных данных и, например, соответствия требованиям GDPR снимаются путем информирования пользователя о том, для чего собираются данные, и обеспечения соответствия фактического использования заявленной цели сбора данных.
    

#### Тестирование на живых данных

### • Теневое развертывание (Shadow Deployment)

Теневое развертывание предусматривает выкатку модели на ПРОМ-данных и сверку получаемых результатов с результатами какой-либо старой модели.

- Разворачиваем новую модель.
    
- Дублируем трафик на новую модель.
    
- Записываем предсказания, но не отдаем клиенту.
    
- Анализируем, ошиблась ли модель.
    

Недостатком такого подхода является тот факт, что мы удваиваем затраты на эксплуатацию, поскольку используем две модели, одна из которых не используется в проде. Кроме того, мы выкатываем новую модель с опозданием из-за необходимости накопить статистику. Необходимо также учитывать смещение выборки, т. е. эффект, при котором старая и новая модель дают разные рекомендации. Проверить правильность рекомендаций новой модели (selection bias), как и оценить результаты несоответствия рекомендаций старой и новой моделей невозможно.

### • Пробный релиз (Canary Release)

Чтобы исправить недостатки теневого развертывания, можно развертывать модель сразу в ПРОМ (пробный релиз), т. е. направить небольшую часть трафика на модель и попытаться использовать ее в ПРОМ. Это позволяет «поймать» грубые ошибки и отладить модель, поскольку такое применение демонстрирует, как модель ведет себя в промышленности. Пробный релиз полезно использовать перед A/Б-тестом или вместо него, если изменения небольшие.

### • А/Б-тестирование

Для проведения А/Б-тестирования мы разворачиваем в проде две модели: А и Б (новая модель) и направляем на каждую модель рандомную часть трафика, после чего сравниваем полученные результаты.

Проблема этого подхода заключается в рандомизации трафика и недостаточной статистической мощности (т.е. объеме выборки). Даже такие гиганты, как Yandex или Google, жалуются, что не могут провести все эксперименты из-за нехватки трафика (мощности выборки).

![https://www.algorithmicmarketingbook.com/, https://hbr.org/2017/09/the-surprising-power-of-online-experiments](https://habrastorage.org/r/w1560/getpro/habr/upload_files/361/cf9/412/361cf9412f2afc47be2498f543524a37.jpg "https://www.algorithmicmarketingbook.com/, https://hbr.org/2017/09/the-surprising-power-of-online-experiments")

[https://www.algorithmicmarketingbook.com/](https://www.algorithmicmarketingbook.com/), [https://hbr.org/2017/09/the-surprising-power-of-online-experiments](https://hbr.org/2017/09/the-surprising-power-of-online-experiments)

### Чередование (Interleaving)

![https://netflixtechblog.com/interleaving-in-online-experiments-at-netflix-a04ee392ec55](https://habrastorage.org/r/w1560/getpro/habr/upload_files/759/28a/a7b/75928aa7b9fa02ccdbddb47e5e95f30f.png "https://netflixtechblog.com/interleaving-in-online-experiments-at-netflix-a04ee392ec55")

[https://netflixtechblog.com/interleaving-in-online-experiments-at-netflix-a04ee392ec55](https://netflixtechblog.com/interleaving-in-online-experiments-at-netflix-a04ee392ec55)

Чтобы исправить ситуацию, можно использовать модель, которая позволяет демонстрировать результаты одновременно двух моделей, например модель рекомендаций, которая позволяет оценить, рекомендации какой именно модели выберет клиент.

### «Многорукие бандиты»

Последний шаг развития с точки зрения развертывания и тестирования модели — использование «многоруких бандитов». Эта теория появилась в 70-х годах, когда пытались максимизировать прибыль от игровых автоматов, рассчитать и максимизировать ожидаемую математическую вероятность выигрыша на том или ином автомате.

В рамках этого подхода мы разворачиваем в ПРОМ несколько моделей и направляем на них трафик в случайном порядке. Чем лучше работает модель, тем больше трафика на нее направляется. Худшие модели убираются или получают меньший объем трафика. В результате мы получаем больше денег от использования тех моделей, которые показывают лучшие результаты в ПРОМ. Преимуществом такого подхода является быстрое переключение на новую модель, а недостатком — сложность по сравнению с А/Б-тестированием.

При этом, проводя A/Б-тестирование или используя подход многорукого бандита, мы работаем с моделью на своей клиентской базе. Благодаря этому мы начинаем сразу эксплуатировать модель, не дожидаясь статистики или результатов, поскольку за это время модель уже может «протухнуть» и ее нельзя будет использовать в производстве.

При использовании данного подхода проводится оценка, какая из моделей (из развернутых в проде) в процессе эксплуатации приносит больше денег. Именно на эту модель начинаем направлять больше клиентов.

### Deploy и промышленная эксплуатация

После тестирования релиза начинается развертывание и промышленная эксплуатация, в этот момент наша идея начинает приносить какие-то реальные деньги и давать свои плоды. 

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/245/eed/273/245eed273626b7a1d6f119366d023358.png)

### Сопровождение

Последний, но не менее важный этап, — сопровождение.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/2fc/4fc/519/2fc4fc51932d95313383899cf3193111.png)

Именно на этом этапе дата-сайентистам необходимо мониторить риск эксплуатируемой модели.

- Из-за изменения среды и предпочтений качество предсказаний может со временем деградировать (сдвиг данных). Необходимо мониторить смещение качества, т.к. эксплуатировать в ПРОМ некачественную модель бессмысленно.
    
- Нужно уметь определять критические пороги для метрик качества входящих данных и самой модели, а также иметь механизмы взаимодействия с командой дата-сайентистов, которые будут заново обучать существующие и создавать новые модели для данного домена.
    

С учетом изменений проводится дообучение или перемоделирование.

- Возможно переобучение модели по запросу. Например, инженеры, отслеживающие качество модели, заметив деградацию качества, направляют запрос в DS-отдел, который выполняет переобучение модели.
    
- Более результативным вариантом является обучение модели по расписанию с учетом сроков возможной деградации. В этом случае дата-сайентисты делают работу, которую можно автоматизировать.
    
- Следующим этапом развития этого процесса является автоматизация оценки качества эксплуатируемой модели и автоматизация обучения. Например, обучение может начинаться по какому-либо триггеру, такому как время, качество модели или накопленный объем исходных данных.
    

### Выводы

Область MLOps является еще достаточно молодой, и в ней до сих пор не существует единого золотого стандарта. На данный момент в MLOps присутствует множество разрозненных инструментов, покрывающих только часть проблем, и очень мало готовых End2End решений, особенно в open-source. Проблематика ее развития остается актуальным и перспективным направлением в Data Science. Будем надеяться, что и в этой области знаний в скором времени появятся свои best practices.