---
tags:
  - data
data_type:
  - DS
link: https://habr.com/ru/companies/tbank/articles/832580/
source: habr
author: TBank
company: TBank
---
Data-driven-привет! 👋 Мы — [Алексей](https://habr.com/ru/users/ALinML/), кроссейл-дата-аналитик, и [Александр](https://habr.com/ru/users/kossandro/), ML-исследователь-разработчик, — объединились, чтобы поделиться нашим алгоритмом машинного обучения по предсказанию клиентского негатива от маркетинговых коммуникаций. 

Слишком навязчивые, нерелевантные или просто несвоевременные маркетинговые касания могут ухудшать пользовательский опыт и вызвать негатив клиентов — от тихого раздражения до заявления в ФАС в ответ на излишний маркетинг. 

В статье мы подробно рассмотрим общую концепцию response-модели, а также технические аспекты ее стратегии обучения, которая показала статистически значимое уменьшение негатива от маркетинга на боевом A/B-тесте. 

## Предпосылки ML-решения

Будем считать крайним негативом и целевой метрикой отказ клиента от маркетинговых коммуникаций через обращение на линию поддержки. 

Работа с таким негативом содержит огромное количество инсайтов: подчеркивает объективные проблемы бизнес-линий, выявляет потребности и боли наших клиентов.

Интересно рассмотреть важность работы над уменьшением негатива и с точки зрения экономики. Многие компании занимаются исследованием негатива для решения явных проблем бизнеса, но не все начинают работать над улучшением клиентского опыта с точки зрения максимизации долгосрочной прибыли — LTV. 

Мы пониманием необходимость развития подходов по улучшению опыта клиентов, и есть две причины использовать именно ML-решение. 

**Нет простых правил,** учитывающих события, приводящие к негативу. Клиенты негативят по разным и независимым друг от друга причинам: из-за частоты коммуникаций, некорректного таргетинга или неподходящего времени.

Есть факторы, которые не зависят напрямую от инициаторов коммуникаций, — характеристики пользователей: пол, возраст, характер общей обращаемости клиента на линию поддержки. Поэтому нам нужны комплексные правила, учитывающие большой спектр характеристик нашего клиента, чтобы ответить на вопрос: стоит ли отправлять ту или иную коммуникацию прямо сейчас?

**Используем 4D-подход.** Перед работой над совокупным алгоритмом мы собрали все технические метрики негатива, визуализировали и изучили динамику их изменения, определили существующие временны́е закономерности. Теперь мы можем провести разведочный анализ данных (exploratory data analysis, EDA) и предсказать факт негатива. То есть мы обладаем всеми необходимыми вводными для построения модели.

## Концепция обучения модели

Атрибутивные правила — это способ определения отклика модели, который формируется путем выбора точки касания и временно́го окна. 

**Вопрос: что считать касанием?** 

При промотировании маркетинга мы можем контролировать только покрытия, то есть только отправки. Конверсии в целевое действие также обычно считаются относительно базы промотирования. 

С другой стороны, включать в response именно отправки некорректно с точки зрения зашумленности событий: не все отправки будут доставлены клиентам. В нашей последней версии модели мы остановились на response-логике через отклик на доставку или показ в соответствующем канале промотирования.

**Вопрос: какое время закладывать на ожидание отклика?**

Коммуникации делятся на три основных типа: 

- маркетинг, то есть рекламные предложения;
    
- немаркетинговые сервисы, например о доставке банковской карты;
    
- информационные, например об изменениях в условиях банковских продуктов. 
    

Рассмотрим статистику попадания в заданное окно отказов после маркетинговых пушей или СМС за шесть месяцев. В день получения или на следующий день отказываются 26% пользователей, далее следует длинный хвост распределения. Но увеличение окна атрибуции до значительных интервалов не имеет смысла. Очевидно, что в какой-то момент клиенту (из хвоста распределения) перестали присылать маркетинговые пуши/СМС, но он в какой-то момент все равно отказался от них по иным причинам. 

![Распределение числа событий (Oy) от числа дней (Ox) до отказа (после касания): слева — марк. пуш/СМС; справа — любые касания в любом канале](https://habrastorage.org/r/w1560/getpro/habr/upload_files/c19/c31/8b6/c19c318b6fec05404ec77f0eb23edae7.png "Распределение числа событий (Oy) от числа дней (Ox) до отказа (после касания): слева — марк. пуш/СМС; справа — любые касания в любом канале")

Распределение числа событий (Oy) от числа дней (Ox) до отказа (после касания): слева — марк. пуш/СМС; справа — любые касания в любом канале

Если считать касания в других каналах и с любым посылом, в день получения или на следующий день отказов уже 64% от всех отказов за полгода. 

Результаты наталкивают на два вывода:

- Отдельная модель предсказания негатива на пуш или СМС с окном атрибуции в два дня будет работать только с четвертью от всего числа отписок.
    
- Часть негатива продуцируется негативом в иных каналах, и, предсказывая пересекающиеся факты отказов (от всего сразу, например), совокупность моделей негатива на разных каналах будет давать лучшее качество.
    

Более того, проведенный анализ наталкивает на предположение о том, что часть клиентов отказываются от пушей или СМС не только из-за маркетинговых уведомлений, но и:

- Из-за немаркетинговых пушей и СМС. Клиент не всегда отличает сервис или информационный посыл от маркетинга.
    
- других цифровых каналов с любым посылом. Триггером для клиента стал иной канал, но, узнав, что можно отказаться от определенного канала, он отключил и пуши/СМС. Типичная формулировка: «Отключить всю рекламу».
    

В подтверждение этого предположения есть много частных примеров при рассмотрении комментариев при отказе:

_«Звонят и назначают встречи после заявки. Хочу отказаться!»_

_«На мою почту приходят рассылки про изменения процентов по вкладам. Хочу отписаться от рассылки!»_

_«Почему мне пришло СМС о переносе встречи в 23:35?»_

Часть таких отзывов несет ценную информацию о потенциальном улучшении стратегии взаимодействия с клиентам, но не с точки зрения маркетинга. У нас есть часть «событий — отказов» от каналов маркетинга по причинам, не связанным с маркетинговым взаимодействием напрямую.

Другой пример непрямого влияния. Триггер в одном канале, а после осознания возможности убрать рекламные плашки в мобильном банке клиент просит отключить еще некие каналы или вовсе отписаться от рекламы во всех каналах сразу. Оценим влияние этого фактора: какая доля отказов происходит именно в том канале, в котором мы последний раз коснулись клиента.

Если доля совпадений низка, это будет означать, что мы можем представить все типы негатива в виде единого обобщения. Некое усредненное негативное действие от отдельного канала до отказа от всего сразу. Если эта доля существенна, есть смысл обучать отдельные модели под каждый канал.

Возьмем простое атрибутивное правило: канал последней доставки, предшествующий отказу, и от каких каналов клиент отказался. Если каналы совпадают, попадание (1), если нет — (0). Оценим долю (1) среди всех событий. Статистика рассчитана за полный 2022 год.

Статистика совпадения каналов отказа и доставок по различным атрибутивным правилам

|   |   |   |   |
|---|---|---|---|
|Last Delivery|   |36 Hours Deliveries|   |
|Доля событий (1), %|Доля событий (0), %|Доля событий (1), %|Доля событий (0), %|
|45,5|54,5|69,9|30,1|

При расчете от последнего касания мы получаем более половины несовпадений (54,5%) — это существенная доля промаха. При атрибуции в 36 часов доля ошибок, естественно, падает, но все еще остается значительной — 30,1%.

Получается, что:

- клиент банка не всегда отличает маркетинговые, информационные и сервисные посылы друг от друга;
    
- клиент банка может отказаться от маркетинга в определенном канале из-за негатива в другом канале.
    

Аналитика показывает, что:

1. Отдельная модель предсказания негатива на пуш или СМС работает только с четвертью от всего числа отписок.
    
2. Доля попаданий негатива с последним маркетинговым каналом контакта — менее 50%.
    

Учитывая вид распределения числа дней от отказа до предыдущего касания, выбрали response-правило для обучающего датасета:

1 — в обучающем датасете это событие наличия отказа от пуша, СМС, чата, мейла или баннерной коммуникации при обращении на линию в течение 36 часов после касания маркетингом в соответствующих каналах промотирования;

0 — в обучающем датасете это событие отсутствия отказа от пуша, СМС, чата, мейла или баннерной коммуникации при обращении на линию в течение 36 часов после касания маркетингом в соответствующих каналах промотирования.

## Единая модель как синергия отдельных решений

Важный акцент как следствие из аналитики выше.

> Если обучать отдельные модели негатива (на мейлы — одна модель, на пуши, СМС и чаты — другая, в баннерах — третья и так далее), каждая такая модель будет работать с ограниченной частью от общего негатива. В структуре негатива 50% от всех отказов — это отписка одновременно от всей рекламы, то есть от всех цифровых каналов промотирования. Показатель сильно меняется со временем

Добиться приемлемого покрытия всех событий негатива возможно только при одновременной работе моделей негатива на всех каналах с определением негативного события «отказ от всего» в каждую из этих моделей.

У решения много неявных технических сложностей, начиная с проблем с разреженными данными и заканчивая просто технической сложностью в поддержании n-моделей. Поэтому обобщение этих типов негатива в один и будет формальным замещением n-моделей. В конечном счете мы предсказываем сам факт обращения клиента на линию с целью отказа от любого маркетинга.

Техническая логика будет выглядеть так:

1. Отбираются за последние 6 месяцев ключи маркетинговых доставок — пуши, СМС, чаты, мейлы и баннерные коммуникации, — от которых можно отказаться. С ограничением числа событий — сэмплированный датасет обучения.
    
2. Отбираются ключи с датами отказов клиентов от любого набора коммуникаций за последние 6 месяцев.
    
3. Среди ключей из п. 2 отбираются только те, у которых была маркетинговая доставка в день отказа или за день до (технически за 36 часов до). Это положительная часть датасета обучения: 1 — события.
    
4. Среди ключей из п. 1 отбираются только те, кто не отказывался через день или в день маркетинговой доставки (через 36 часов после). Это отрицательная часть датасета обучения: 0 — события.
    
5. К датасету обучения подтягиваются все фичи (подробнее в следующем разделе) для последующей передачи на EDA, Feature Engeneering и ML.
    

В итоге получаем сэмплированный набор данных, в котором доля отказов многократно увеличена относительно реальной конверсии в негатив.

Важный аспект обучения любого ML-алгоритма — набор данных. Всего для обучения модели было собрано 787 признаков, из которых 152 — социально-демографические, 636 — признаки поюзерного поведения в различные периоды. Рассмотрим их подробнее:

- 20 признаков — семейство фичей, отражающих реакцию пользователей в каналах историй и чатов;
    
- 6 признаков — виральность, как часто и сколько клиент рекомендовал продукт друзьям/знакомым;
    
- 24 признака — определение продуктового возраста клиента, совокупного и по отдельным основным продуктам банка;
    
- 135 признаков — семейство фичей, отражающих тенденции касаний клиента маркетингом в активных каналах и эмулирующих влияние контактных политик со временем. В том числе среднее время между коммуникациями по каналам, число доставленных, прочитанных и открытых уведомлений и тому подобное;
    
- 135 признаков — аналогично для НЕмаркетинговых коммуникаций в активных каналах;
    
- 252 признака — аналогично для всех типов реактивных каналов привлечения;
    
- 14 признаков — негативные реакции на баннерные коммуникации;
    
- 12 признаков — негативные реакции на email-коммуникации;
    
- 10 признаков — семейство фичей по общей тенденции обращаемости на линию клиентов, не только с темой отказа от рекламы;
    
- 12 признаков — общая активность в приложении банка;
    
- 11 признаков — графа друзей и наличия специальных условий, подписок и прочего.
    

## Обучение и метрики качества ML-решения

Рассмотрим основные этапы обучения модели.

**EDA.** Признаки с относительно высокой корреляцией не удалялись, так как в контексте алгоритмов ансамбля решающих деревьев проблема мультиколлинеарности обладает менее выраженным влиянием. В сравнении с линейными моделями: линейная регрессия или логистическая регрессия.

Часть признаков была удалена по результатам дисперсионного анализа (вариативность признаков). Низковариативные признаки (или с нулевой вариацией, т. е. практически с одинаковыми значениями) были удалены из рассмотрения по результатам EDA.

**Train/Val/Test Split.** [Существует множество стратегий кросс-валидации](https://arxiv.org/pdf/2205.05393) в подобных задачах. Учитывая контекст, схожий с алгоритмами рекомендательных систем, модель не должна обучаться условно на августе и предсказывать июнь, поэтому было принято решение использовать подход time-based split. Последовательно 4 месяца — train, 1 месяц — validation, 1 месяц — test data. При этом метрики качества считались и для отдельных категорий пользователей: среди тех, кто был и в train-, и test-наборах (test’d train — группа), и, наоборот, среди тех, кто попал только в тестовый набор данных (test only — группа).

**LGBM vs XGB vs CatBoost.** Алгоритмы классификации первоначально сравнивались в базовой настройке, то есть без отдельного подбора гиперпараметров (c постфиксом _0). Далее при помощи фреймворка для оптимизации гиперпараметров OPTUNA были выбраны лучшие настройки параметров каждой модели c постфиксом _new. В конечном счете был выбран алгоритм CatBoostClassifier с дополнительной конфигурацией гиперпараметров, тестовый ROC_AUC финально которого составил 0,872 (на группе test).

Распределение ROC_AUC по различным версиям алгоритмов в базовой настройке и с подбором гиперпараметров

|   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|
|Группа|LGB_0|LGB_new|XGB_0|XGB_new|CatBoost_0|CatBoost_new|
|Train|0,908|0,911|0,913|0,887|0,890|0,897|
|Validation|0,853|0,865|0,847|0,857|0,855|0,869|
|Test|0,857|0,868|0,854|0,855|0,859|0,868|
|Test only|0,854|0,866|0,853|0,854|0,858|0,867|
|Test’d train|0,880|0,897|0,882|0,879|0,886|0,893|

## Метрики качества и интерпретация результатов

GPT4 предложил назвать response-модель CatBoostClassifier в финальной настройке гиперпараметров аббревиатурой NOPE: Negative Opinion Prediction Estimation, «расчет прогноза отрицательного мнения». Это название и закрепилось за разработанным алгоритмом. 

Покажем распределение некоторых качественных показателей итогового решения на тестовых данных при обучении.

![а — гистограмма распределения скоров модели; b — ROC Curve; c — Precision-Recall vs Threshold Cur](https://habrastorage.org/r/w1560/getpro/habr/upload_files/ebf/4a5/a85/ebf4a5a856f3ebd6ad48170efd0c7a55.png "а — гистограмма распределения скоров модели; b — ROC Curve; c — Precision-Recall vs Threshold Cur")

а — гистограмма распределения скоров модели; b — ROC Curve; c — Precision-Recall vs Threshold Cur

Рассчитали показатели [SHAP values](https://habr.com/ru/companies/ods/articles/599573/) и [Feature importance](https://catboost.ai/en/docs/concepts/fstr?ysclid=lrhl87ytva299886972), чтобы оценить важность признаков или почему алгоритм принимает то или иное решение относительно конкретного пользователя в определенный день. Обученные модели в одних и тех же гиперпараметрах, но с разными random_state, приводят к небольшим различиям в результатах. Особенно это касается интерпретации важности признаков. 

Для результатов семи отдельных NOPE-моделей (с различными состояниями random_state) и топ-10 признаков были получены средние нормированные значения отдельно для SHAP и FI (соответствующие rank FI и rank SHAP), а также средние между ними — итоговые ранги важности (RN). Для результатов семи отдельных NOPE-моделей и с различными состояниями random_state и топ-10 признаков получили средние нормированные значения отдельно для SHAP и FI. Глядя на распределение SHAP values, не стоит считать первый признак буквально наиболее важным. Скорее он один из важных, но более достоверное первенство по важности находится в таблице со значением RN = 1.

Распределение значимости топ-10 признаков алгоритма

|   |   |   |   |   |   |
|---|---|---|---|---|---|
|**Признак**|**avg** **FI**|**avg** **SHAP**|**rank** **FI**|**rank** **SHAP**|**RN**|
|Среднее число дней между обращениями (за последние 60 дней)|7,077445|0,142448|1|9|1|
|Возраст (биологический)|5,560440|0,145044|2|8|2|
|Число обращений за последние 14 дней|4,093130|0,138456|4|10|3|
|Возраст (продуктовый)|2,745346|0,177486|5|4|4|
|Число обращений за последние 30 дней|4,325800|0,123765|3|15|5|
|Статус наличия дебетовой карты банка|1,231935|0,206564|20|1|6|
|Пол|1,389538|0,192425|16|2|7|
|Число маркетинговых коммуникаций, полученных за последние 7 дней|2,687448|0,128252|6|14|8|
|Статус пользования сервисом «Топливо»|0,333589|0,191593|48|3|9|
|Статус наличия кредитной карты банка|1,074774|0,160095|24|5|10|

С точки зрения прозрачности результатов для бизнеса подготовили инструменты с возможностью визуализации градации важности признаков для алгоритма в целом и для отдельно взятого клиента.

![a — SHAP WATERFALL, график для интерпретации значимости значений признаков для отдельного взятого пользователя; b — SHAP VALUES для оценки распределения важности значений признаков в целом на всех пользователях](https://habrastorage.org/r/w1560/getpro/habr/upload_files/898/4b1/911/8984b191162a9d2bc059f1c99f7a13fc.png "a — SHAP WATERFALL, график для интерпретации значимости значений признаков для отдельного взятого пользователя; b — SHAP VALUES для оценки распределения важности значений признаков в целом на всех пользователях")

a — SHAP WATERFALL, график для интерпретации значимости значений признаков для отдельного взятого пользователя; b — SHAP VALUES для оценки распределения важности значений признаков в целом на всех пользователях

Пол — важный и хорошо распределенный признак. В данном случае он не раскрашен, так как это категориальный признак. И это позволяет не указывать однозначно, какой пол в большей степени подвержен проявлению негатива. 

К примеру, хорошо сепарированный признак — продуктовый возраст: высокие значения (клиенты, которые долго используют продукты банка) лояльны, что влияет на уменьшение скора негатива, клиенты новые или еще без оформленного продукта повышают скор проявления негатива. Обратная тенденция заметна, например, на признаке числа коммуникаций за последние 7 дней: очевидно, чем выше коммуникационная нагрузка на пользователя, тем больше вероятность его обращения в поддержку с негативом, и наоборот.

В следующей статье мы подробно разберем дизайн A/B-теста in production, поделимся статистически значимыми результатами, а также разберем все нюансы теста.