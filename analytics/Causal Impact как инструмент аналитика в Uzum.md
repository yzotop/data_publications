---
link: https://habr.com/ru/companies/uzum/articles/817473/
tags:
  - data
source: habr
data_type:
  - Analytics
company: Uzum
author: Uzum
---
Всем привет. Меня зовут Кирилл, я маркетинговый аналитик в Uzum. В этой статье я хочу поделиться с вами практическими примерами, которые демонстрируют реальную ценность методов причинно-следственного анализа. Отдельно расскажу, как библиотека CausalImpact помогает в решении бизнес-задач.

#### Проблема

Мы, как и многие маркетплейсы, постоянно работаем над оптимизацией операционной нагрузки и улучшением финансовых показателей. Одной из главных задач является обеспечение баланса между доступностью цен для наших покупателей и рентабельностью бизнеса. В качестве одного из решений предложили внедрить минимальную стоимость заказа. Эта мера была необходима, потому что логистические издержки при доставке мелких заказов оказываются слишком высокими, делая такие заказы убыточными для компании. Например, затраты на упаковку, транспортировку и обработку небольших заказов могут превышать их доходность, что негативно сказывается на общей финансовой картине. Введение минимальной стоимости заказа помогает нам сократить логистические издержки и повысить эффективность операций. Мы стремимся сохранять цены доступными для наших покупателей, чтобы минимальная стоимость заказа была разумной и не отпугивала клиентов.

Это изменение ввели без проведения эксперимента с контрольной группой, сразу на всю аудиторию маркетплейса, что затруднило оценку его воздействия на ключевые метрики. Мы поступили так, поскольку маркетплейс уже испытывал проблемы с экономикой, а в преддверии новогодних распродаж нагрузка по заказам росла и нужно было решать ASAP. Внедрение новых фич без A/B-экспериментов порождает вопросы о том, какие изменения в метриках являются результатом нововведений, а какие можно объяснить внешними факторами или естественной волатильностью данных.

![Аналитики, когда изменения катятся без A/B-теста.](https://habrastorage.org/r/w1560/getpro/habr/upload_files/59d/db9/13a/59ddb913a8a2e1673ebb37fb1d524900.png "Аналитики, когда изменения катятся без A/B-теста.")

Аналитики, когда изменения катятся без A/B-теста.

Когда нет контрольной группы для прямого измерения воздействия, можно использовать методы моделирования условий «что было бы, если». Это позволяет найти причинно-следственные связи, опираясь на имеющиеся данные, и делать обоснованные выводы о влиянии изменений.

Эту методологию можно описать через концепцию лестницы доказательств, где методы моделирования, такие как CausalImpact, относятся к квазиэкспериментальным исследованиям. Эти методы, хотя и менее надежны, чем рандомизированные контролируемые исследования, помогают анализировать влияние изменений, когда проведение прямых экспериментов невозможно.

#### Метод

Когда изменения раскатываются без эксперимента, мы обращаемся к методу байесовских структурных временных рядов, который позволяет оценить, как развивались бы события, если бы изменение не было внедрено. Этот метод особенно полезен для анализа влияния определенных действий или нововведений на временные ряды данных, когда прямой контроль над экспериментальными и контрольными группами отсутствует.  
  
Вот как это работает простыми словами:

- Модель анализирует временные ряды до момента введения изменения, чтобы определить сезонность, тренды и циклические колебания
- В модель вводят ковариаты для учета внешних факторов, которые могут повлиять на основной временной ряд. Это помогает улучшить точность прогнозов, поскольку ковариаты предоставляют дополнительную информацию о возможных внешних воздействиях.
- CausalImpact ансамблирует получившиеся прогнозные модели, чтобы учесть различные возможные сценарии и уменьшить неопределенность в прогнозах, и генерирует контрфактический прогноз — предсказание того, что могло бы произойти, если бы изменения не были введены.
- После внедрения изменений фактические данные сравниваются с контрфактическим прогнозом. Разница между ними показывает причинно-следственное воздействие внедренной меры.
    
То есть CausalImpact создает синтетический контрольный временной ряд, который представляет собой ансамбль прогнозов на основе исторических данных до изменения и информации от ковариат. Этот подход позволяет нам оценить эффект от внедрения изменения в условиях, когда невозможно провести контролируемый эксперимент.

#### Технический подход

Итак, есть проблема: раскатили фичу без теста. Нам интересно, как это повлияло на количество заказов и GMV. Нам важно не уронить GMV и снизить количество заказов с низким чеком. Есть метод для оценки таких изменений. Как мы действуем:

- Собрали данные: DAU, количество заказов, GMV по дням. Будем смотреть, как изменилось количество заказов и GMV после введения изменений. DAU будем использовать в качестве ковариаты.
    
- С помощью библиотеки CausalImpact создали модель байесовских структурных временных рядов, включая данные по заказам, GMV и DAU.
    
- Сравнили фактические показатели после введения минимальной стоимости заказа с прогнозными значениями, полученными моделью для сценария «что было бы, если бы изменений не произошло».
    

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/b43/d56/66b/b43d5666b609490ae9751ddcb2e44e54.png)

### Как выбирать ковариату

Ковариата — это переменная для учета воздействия внешних факторов, не связанных напрямую с внедренным изменением, влияющая на анализируемые показатели. Мы выбрали метрику DAU. Предполагается, что введение минимальной суммы заказа не должно напрямую влиять на количество активных пользователей. Кроме того, DAU хорошо коррелирует с заказами и GMV, что позволяет модели лучше адаптироваться к естественным колебаниям в данных и обеспечивает более точное моделирование влияния введенных изменений.

Выбор ковариат критически важен при построении любой модели причинно-следственного анализа. Они помогают модели «понять» контекст данных, учитывая внешние факторы, которые могут влиять на анализируемые показатели. Кроме того, можно было бы добавить ковариаты, отражающие маркетинговую активность (например, расходы на рекламу), что также может влиять на покупательское поведение.

При выборе ковариат следует избегать включения метрик, которые могут слишком сильно коррелировать друг с другом, поскольку это может привести к мультиколлинеарности и затруднить интерпретацию результатов.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/788/315/1ca/7883151ca83584e4900eff183c41013d.png)

Ниже приведена упрощенная структура кода для демонстрации подхода к анализу. Основываясь на предоставленных данных, модель CausalImpact анализирует изменения в метрике «orders» с учетом ковариаты «dau», разделяя наблюдаемый период на «до» и «после» введения минимальной стоимости заказа.

```
#импортируем библиотеку:
from causalimpact import CausalImpact

#у нас есть датафрейм с датами в индексе и колонками 'orders' и 'dau'
orders = data[['orders', 'dau']]

#задаем период до изменений и после
pre_period = ['2023-09-01', '2023-12-06']
post_period = ['2023-12-07', '2023-12-30']

#все волшебство в одной строчке кода 
#если у вас данные в днях, можно указать period=7
ci = CausalImpact(orders, pre_period, post_period, model_args={ 'period': 7})
```

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/2f5/ce0/9c3/2f5ce09c39651dbefdf47a3e81af4d6a.png)

```
# далее можно вывести вывод
print(ci.summary())
```

Тут будет представлен результат: было ли изменение, какое, доверительные интервалы.

```
# можно построить графики
ci.plot(figsize=(15, 12))
```

Будет выведено три графика:

![Фактические значения и предсказания.](https://habrastorage.org/r/w1560/getpro/habr/upload_files/f2d/a78/ac4/f2da78ac452f208c7fb8cf53cc2489be.png "Фактические значения и предсказания.")

Фактические значения и предсказания.

Фактическое количество заказов — «y» (черная линия) и предсказанное моделью количество заказов (синяя пунктирная линия). Это позволяет визуально сравнить реальные данные после введения изменений с прогнозируемыми значениями, которые модель рассчитала бы, если бы изменения не были введены. Широкая фиолетовая область вокруг предсказанной линии указывает на доверительный интервал прогноза, отражая неопределенность модели.

![Разница между фактическими значениями и предсказанием.](https://habrastorage.org/r/w1560/getpro/habr/upload_files/2da/f73/d00/2daf73d007f9e02305dde38c42d75b1a.png "Разница между фактическими значениями и предсказанием.")

Разница между фактическими значениями и предсказанием.

График показывает разницу (эффект) между фактическими значениями и контрфактическими предсказаниями на каждый момент времени после введения изменения. Положительные значения свидетельствуют о том, что фактические данные выше предсказанных (эффект больше нуля), в то время как отрицательные значения указывают на противоположный эффект. Как и на первом графике, фиолетовый коридор представляет доверительный интервал.

![Кумулятивный эффект.](https://habrastorage.org/r/w1560/getpro/habr/upload_files/787/6e8/f4e/7876e8f4eb933a2c15605e6025dbf02b.png "Кумулятивный эффект.")

Кумулятивный эффект.

На графике представлен кумулятивный эффект введенной меры. Это интегральная оценка того, как воздействие изменения накапливается день за днем. Здесь также показан доверительный интервал, который дает представление о надежности накопленного эффекта. Кумулятивный эффект особенно полезен для понимания общего влияния изменения на интересующую метрику за анализируемый период.

Эти графики в совокупности дают полное представление о воздействии введенной минимальной стоимости заказа на количество заказов, позволяя нам не только оценить моментальное изменение, но и увидеть общую динамику эффекта во времени. То же самое было проделано c GMV.

#### Выводы по этому эксперименту

Анализ показал, что GMV не снизился, что подтверждает отсутствие сильного влияния введения минимальной суммы заказа на общую стоимость продаж. Увеличение среднего чека компенсировало снижение количества низкочековых заказов, сохранив уровень GMV. Пользователи стали лучше планировать свои покупки, объединяя мелкие заказы в один большой, что также способствовало снижению логистических издержек. Таким образом, введение минимальной суммы заказа оказалось успешным в краткосрочной перспективе, уменьшив затраты на логистику и сохранив общую экономическую эффективность маркетплейса. Забегая вперед, позднее мы провели обратный A/B-тест, чтобы проверить результаты модели, и продолжили мониторинг долгосрочных тенденций, так как изменение в потребительских паттернах, таких как частотность покупок и удержание, может проявиться со временем, о чем расскажем в следующих материалах. 

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/c0d/1f2/a9e/c0d1f2a9e199070529b50cb48831db42.png)

#### Еще эксперименты

Метод CausalImpact действительно универсален и находит широкое применение в нашей работе. Мы используем его не только для оценки маркетинговых кампаний, но и для анализа различных бизнес-решений.

При оценке маркетинговых кампаний часто возникает проблема, когда после обучения кампании начинают показываться «похожим» людям, что затрудняет проведение честного эксперимента. В таких ситуациях метод CausalImpact становится незаменимым инструментом. Мы ограничиваем кампании по регионам, находим регионы с аналогичной динамикой и запускаем кампанию в одном из них, используя другой регион в качестве ковариаты. Это позволяет нам получить объективную оценку эффективности и сравнить с результатами, полученными с помощью расчета эффективности кампаний по атрибуции.

Еще один пример использования метода — анализ влияния открытия новых пунктов выдачи заказов (ПВЗ). Открытие нового ПВЗ может оказывать негативное влияние на соседние пункты. Мы применяем CausalImpact, чтобы оценить это влияние: ПВЗ, рядом с которым открылся новый пункт, — исследуемый объект, а в качестве ковариаты используется третий ПВЗ, который по своим показателям схож, но достаточно далеко и не подвергся влиянию. А затем мы можем оценить общий экономический рост района благодаря открытию ПВЗ, взяв ковариатой другой район.

#### Заключение

Сложные инструменты анализа, особенно в условиях, когда невозможно использовать традиционные экспериментальные подходы, стали как никогда просты благодаря современным технологиям и методам, таким как CausalImpact. Эти инструменты позволяют нам проводить глубокий анализ и получать ценные инсайты, даже когда проведение классических A/B-тестов невозможно или затруднено. Мы можем принимать обоснованные решения на основе данных и эффективно оптимизировать бизнес-процессы.  
В Uzum мы всегда рады талантливым специалистам, стремящимся перевернуть мир больших данных и способным работать с передовыми аналитическими методами. Мы предлагаем динамичную и инновационную среду, где каждый сотрудник может реализовать свой потенциал, работая над интересными и значимыми проектами. Присоединяйтесь к нам, чтобы вместе создавать будущее анализа данных и делать нашу экосистему лучше!

Ссылка на оригинальную статью, для тех кто хочет более глубокого понимания темы:

[https://storage.googleapis.com/gweb-research2023-media/pubtools/pdf/41854.pdf](https://storage.googleapis.com/gweb-research2023-media/pubtools/pdf/41854.pdf)