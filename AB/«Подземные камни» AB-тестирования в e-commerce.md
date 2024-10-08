---
tags:
  - data
data_type:
  - AB tests
link: https://habr.com/ru/companies/retailrocket/articles/801667/
company: RetailRocket
source: habr
author: RetailRocket
---
Любой полезный бизнесу продукт меняется со временем: появляются новые функции, улучшаются старые. Возникает потребность оценить влияние таких изменений на пользователей продукта. Необходимо проверить, нет ли ошибок в реализации новой функциональности и справляется ли она с поставленными задачами. 

Первое, что хочется сделать — сравнить показатели работы продукта до внесения изменений и после. Но в таком случае нельзя утверждать, что разница в показателях обусловлена только новой функциональностью, так как на состояние продукта в любой момент времени может повлиять любой внешний фактор. Поэтому принято прибегать к контролируемым рандомизированным экспериментам, которые также называют А/Б-тестами. В том числе и для товарных рекомендаций в e-commerce.

Мы в Retail Rocket Group больше 10 лет предоставляем рекомендации онлайн-магазинам и сталкиваемся с А/Б-тестами. За это время у нас накопился обширный опыт, которым хотелось бы поделиться. Мы используем А/Б-тесты, как для решения своих задач, так и по запросам наших клиентов, и часто видим:

- ложные ожидания от А/Б-тестов;
    
- использование А/Б-тестов там, где они не приносят пользу; 
    
- ошибки в дизайне экспериментов;
    
- неполную интерпретацию результатов.
    

Область А/Б-тестов обширна, что подтверждает большое количество статей и книг по данной тематике. Например, хорошим источником является книга «[Доверительное А/Б-тестирование](https://dmkpress.com/catalog/computer/software_development/978-5-97060-913-2/)» от ведущих специалистов по экспериментам в Google, LinkedIn и Microsoft. В этой статье мы раскроем небольшую часть аспектов, связанных с А/Б-тестами. Мы выбрали ту часть, которая кажется нам наиболее недооценённой и игнорирование которой приводит к проблемам. 

### 1. Отсутствие А/А-тестов

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/14d/6c6/0cb/14d6c60cb65ac4896838d61f7cd6c3b6.png)

Одним из важных аспектов при организации А/Б-тестов является проведение А/А-тестов. А/А-тест повторяет будущий А/Б-тест с той лишь разницей, что пользователям всех сегментов показывается одинаковый контент. Так как пользователям показывается одно и то же, то и между сегментами А/А-теста не должно быть значимых различий. В этом случае можно запускать А/Б-тест, иначе необходимо найти причину их возникновения. 

Причинами возникновения различий могут быть проблемы с делением трафика, с недостаточностью данных или с аномалиями. Эти проблемы необходимо решать, в противном случае они могут исказить результаты будущих А/Б-тестов. А значит, будут приняты неверные решения, которые не только не принесут пользу, но и могут нанести вред бизнесу.

Мы часто сталкиваемся с недооценкой вероятности возникновения проблем ([Глава 19.1](https://dmkpress.com/catalog/computer/software_development/978-5-97060-913-2/)), которые способны обнаружить А/А-тесты, и последствий, к которым они могут привести. Далее мы приведём несколько примеров из нашего опыта.

#### Обнаружение ошибки в делении трафика

Иногда интернет-магазины решают проводить А/Б-тесты самостоятельно. И в отдельных случаях в результатах предварительных А/А-тестов нами наблюдались отклонения в пользу одного из сегментов. Зачастую можно было заметить, что в один из сегментов попадало больше пользователей, чем в другой. В итоге выяснялось, что проблема возникала из-за неправильного деления трафика.

Проведение А/А-тестов в таких случаях позволяло избежать получения ошибочных результатов в дальнейших А/Б-тестах. А также позволяло избежать издержек, связанных с проведением повторных А/Б-тестов, в случае, если бы проблема обнаружилась позже. 

Ошибки с делением трафика связаны с показателем несоответствия коэффициента выборки (sample ratio mismatch, SRM). Подробнее с тем, к чему может привести SRM и как с этим бороться, можно ознакомиться в [главе 21](https://dmkpress.com/catalog/computer/software_development/978-5-97060-913-2/).

#### Малое количество трафика увеличивает вероятность расхождения

Расхождения в А/А-тестах [часто происходят](https://dl.acm.org/doi/10.1145/3534678.3539160) на страницах с малым количеством трафика. Мы убедились в этом на собственном опыте, проведя сотни А/А-тестов.

В таблице ниже представлены характерные для данной проблемы результаты одного из А/А-тестов. Видно, что сегмент Б значимо лучше сегмента А по метрике «конверсия» с достаточно сильным изменением метрики.

|**Сегмент**|**Количество пользователей**|**Метрика**|**Значение метрики**|**Значимость**|**Изменение метрики**|
|---|---|---|---|---|---|
|A|2760|Конверсия|0.3196|||
|Б|2692|Конверсия|0.3592|0.0010|+11.04%|

В данной статье мы ещё коснёмся этого вопроса в разделе про недостаточную мощность.

#### Расхождения вследствие аномальных пользователей

Расхождения в А/А-тестах могут происходить из-за отдельных аномальных пользователей. Для метрик, связанных с выручкой — это пользователи, которые совершают заказы на сумму, в разы превышающую суммы заказов любого из остальных участников эксперимента. Мы сталкивались с тем, что такое происходило, например, из-за ошибки при оформлении заказа.

В таблице ниже представлены характерные для данной проблемы результаты одного из А/А-тестов. В данном случае в сегмент Б попал аномальный пользователь. 

|**Сегмент**|**Количество пользователей**|**Метрика**|**Значение метрики**|**Значимость**|**Изменение метрики**|
|---|---|---|---|---|---|
|A|611874|Средняя выручка на пользователя|1484.95|||
|Б|610625|Средняя выручка на пользователя|1551.91|0.0186|+4.51%|

Подобные проблемы часто возникают с метриками «средняя выручка на пользователя» и «средний чек». В качестве решения мы предлагаем использовать метрики, менее чувствительные к аномалиям.

#### Влияние сильных аномалий на долю расхождений

Как известно, перед проведением эксперимента выбирают вероятность, с которой может появиться ошибка — результат, которого на самом деле нет. Эту вероятность обычно называют уровнем значимости. Любое расхождение в А/А-тесте является следствием такой ошибки. Это означает, что если провести большое количество А/А-тестов, то доля расхождений должна быть приблизительно равна заданному уровню значимости, например 0.05. Так, если провести 1000 А/А-тестов, то с уровнем значимости 0.05 должно разойтись около 50.

В трафике некоторых интернет-магазинов присутствуют настолько сильные аномалии, что доля таких расхождений существенно увеличивается. Достоверно определить это в результате единичного А/А-теста не получится. Запустить большое количество А/А-тестов не представляется возможным. Но можно провести симуляцию множества А/А-тестов ([Глава 19.2](https://dmkpress.com/catalog/computer/software_development/978-5-97060-913-2/)). Для этого необходимо:

1. Поделить трафик магазина случайным образом на сегменты;
    
2. Посчитать значимость расхождения между этими сегментами;
    
3. Повторить такую процедуру многократно, используя каждый раз новое случайное деление трафика;
    
4. Построить гистограмму распределения значимости и посмотреть, равномерно ли оно.
    

На рисунке ниже приведён пример гистограммы для трафика без аномалий.

![Распределение значимости для трафика без аномалий](https://habrastorage.org/r/w1560/getpro/habr/upload_files/1bf/06b/c77/1bf06bc77e3cb376aa1b081ea4f607fe.png "Распределение значимости для трафика без аномалий")

Распределение значимости для трафика без аномалий

Для трафика с сильными аномалиями гистограмма может выглядеть следующим образом:

![Распределение значимости для трафика с сильными аномалиями](https://habrastorage.org/r/w1560/getpro/habr/upload_files/041/b2c/98c/041b2c98c0ab84bf9f3a1aebc848181b.png "Распределение значимости для трафика с сильными аномалиями")

Распределение значимости для трафика с сильными аномалиями

Видно, что распределение значимости далеко от равномерного. В таких случаях мы не рекомендуем проводить А/Б-тесты, не решив проблему. В частности, можно использовать метрики, менее чувствительные к аномалиям.

Проводить подобные симуляции можно на любом историческом трафике. Но стоит отметить, что исследование трафика является нетривиальной задачей, для решения которой требуется специалист, наличие которого не всегда можно обеспечить. Кроме того, здесь не исключены ошибки ввиду человеческого фактора. Поэтому надежнее будет запустить А/А-тест на настоящем трафике и обнаружить влияние аномалий практическим путем.

### 2. Недостаточная мощность выбранной для А/Б-тестов метрики

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/4b5/577/3d4/4b55773d4e0879eeb71a5af62f4b95d3.png)

Ещё одной важной процедурой при проведении А/Б-тестов является оценка мощности метрик. Мощность метрики — это вероятность увидеть значимый результат при условии, что есть польза (или вред) от новой функциональности. Важно отметить, что анализ мощности происходит до запуска эксперимента. Чтобы посчитать мощность метрики, необходимо взять исторический трафик, зафиксировать ожидаемое изменение метрики и использовать формулу для этой метрики. Обычно [80%](https://en.wikipedia.org/wiki/Power_of_a_test#Interpretation) достаточно для проведения теста.

#### К чему приводит недостаточная мощность

Низкая мощность ведёт к тому, что даже если тестируемая функциональность оказывает положительное влияние, в результатах теста этого заметно не будет. Будут впустую потрачены ресурсы и полезная функциональность не будет использована.

Дополнительной опасностью является то, что чем ниже мощность, тем выше вероятность, что любой [значимый результат окажется ложным](https://f.hubspotusercontent00.net/hubfs/215600/qubit-research-ab-test-results-are-illusory-1.pdf). Это приведёт к тому, что бесполезная функциональность будет использоваться и на её поддержку будут тратиться ресурсы. Более того, может быть использована функциональность, которая на самом деле хуже исходного варианта. 

Проиллюстрируем возникновение проблемы на примере. Допустим, мы проводим 100 А/Б-тестов, в 10 из которых есть реальное влияние на метрику. Если выбран порог значимости в 5% и мощность каждого из экспериментов 80%, то из 10 тестов влияние на метрику мы должны обнаружить в 8. Ещё 5 тестов окажутся с ложными положительными результатами из-за выбранного порога значимости. Таким образом, мы получим 8 + 5 = 13 тестов со значимыми результатами, 5 / 13 = 38.5% из которых будут ложными. В случае более низкой мощности, например, в 20% ложными окажутся 5 / 7 = 71.4% результатов, то есть почти вдвое больше.

Кроме того, изменение метрики в экспериментах с низкой мощностью сильно завышено. В том числе, если это ложный результат. Об этом пишут эксперты по тестированию из Microsoft и Airbnb в своей [статье](https://dl.acm.org/doi/10.1145/3534678.3539160). То есть если вы получили значимый результат в А/Б-тесте с малой мощностью, то изменение метрики в нём будет сильно отличаться от того, что есть в реальности.

Вернёмся к [примеру](https://habr.com/ru/companies/retailrocket/articles/801667/#:~:text=%D0%9C%D0%B0%D0%BB%D0%BE%D0%B5%20%D0%BA%D0%BE%D0%BB%D0%B8%D1%87%D0%B5%D1%81%D1%82%D0%B2%D0%BE%20%D1%82%D1%80%D0%B0%D1%84%D0%B8%D0%BA%D0%B0%20%D1%83%D0%B2%D0%B5%D0%BB%D0%B8%D1%87%D0%B8%D0%B2%D0%B0%D0%B5%D1%82%20%D0%B2%D0%B5%D1%80%D0%BE%D1%8F%D1%82%D0%BD%D0%BE%D1%81%D1%82%D1%8C%20%D1%80%D0%B0%D1%81%D1%85%D0%BE%D0%B6%D0%B4%D0%B5%D0%BD%D0%B8%D1%8F) с малым количеством трафика, который мы привели выше:

- Метрика изменилась на 11%, хотя от рекомендаций мы ожидаем 3-5%. 
    
- Мощность метрики для изменения в 5% составляет [35.2%](https://www.mycompiler.io/view/4gKVh0p7HIr), что очень мало.
    
- Сегменты значимо отличаются в А/А-тесте, хотя не должны.
    

Это говорит о том, что А/Б-тест в данном случае проводить нельзя или по крайней мере нельзя оценивать его результаты по выбранной метрике.

Существуют способы увеличения мощности такие, как [фильтрация аномалий](https://habr.com/ru/companies/avito/articles/571094/#outliers-in-data), [cuped](https://exp-platform.com/Documents/2013-02-CUPED-ImprovingSensitivityOfControlledExperiments.pdf), [стратификация](https://www.kdd.org/kdd2016/papers/files/adp0945-xieA.pdf). Но во-первых, методы, которые усложняют логику обработки результатов,  вызывают недоверие у клиентов. Во-вторых, добиться достаточного уровня мощности для некоторых метрик, таких как «средняя выручка на пользователя», с их применением чаще всего не получается.

#### Выбор наиболее мощных метрик

При тестировании рекомендаций используется множество метрик. Конечной целью интернет-магазина является увеличение прибыли, которую он получает за всё время работы с пользователем — [LTV](https://en.wikipedia.org/wiki/Customer_lifetime_value). Но поскольку длительный эксперимент является проблемой, в А/Б-тестах рекомендаций смотрят на краткосрочные метрики, изменение которых ведёт к изменению этого показателя. 

При этом чаще всего используются метрики, связанные с заказами, такие как «конверсия», «средний чек» или «средняя выручка на пользователя». Реже используют такие метрики, как «среднее количество просмотренных карточек товаров», «конверсия в пользователя с корзиной» или «среднее количество товаров, добавленных в корзину». Главное отличие метрик с заказами от остальных в том, что они показывают пользу от рекомендаций в денежном эквиваленте. Но вместе с этим зачастую такие метрики имеют более низкую мощность, поэтому на практике могут быть менее полезными. 

В случае конверсии можно было бы достигнуть достаточного уровня мощности за счёт увеличения длительности теста и, как следствие, количества покупателей. Это возможно, потому что мощность конверсии зависит только от количества пользователей и покупателей среди них. Поэтому, кстати, именно расчёт мощности конверсии часто используется в онлайн калькуляторах для А/Б-тестов. 

Но в случае выручки мощность зависит ещё и от разброса. Например, есть магазины, где суммы трат пользователей меняются от 10 тыс до 1 млн в течение эксперимента. Чем выше разброс между суммарными тратами пользователей, тем меньше мощность метрики. Достижения достаточного уровня в таком случае можно не дождаться вовсе. На рисунке ниже изображён график мощности средней выручки на пользователя в зависимости от количества дней проведения эксперимента на трафике одного из наших клиентов.

![Мощность метрики «средняя выручка» в зависимости от длительности эксперимента](https://habrastorage.org/r/w1560/getpro/habr/upload_files/d20/c54/cec/d20c54cecc9cfc5a6297d7479b6fc406.png "Мощность метрики «средняя выручка» в зависимости от длительности эксперимента")

Мощность метрики «средняя выручка» в зависимости от длительности эксперимента

Кроме того, только покупатели магазина влияют на метрики, связанные с выручкой. Но существуют метрики, на которые влияют все посетители и разброс которых ниже. Соответственно эти метрики обладают большей мощностью. При этом они хорошо коррелируют с конечной целью интернет-магазина — увеличением LTV пользователей. Об этом в нашем блоге есть отдельный [цикл статей](https://habr.com/ru/companies/retailrocket/articles/591205/). На рисунке ниже изображён график мощности одной из таких метрик в зависимости от количества дней проведения эксперимента на тех же данных, что и на предыдущем рисунке.

![Мощность метрики «среднее количество просмотренных карточек товаров» в зависимости от длительности эксперимента](https://habrastorage.org/r/w1560/getpro/habr/upload_files/ca3/02d/e0f/ca302de0fa7ea25f634139cdc99c3cd3.png "Мощность метрики «среднее количество просмотренных карточек товаров» в зависимости от длительности эксперимента")

Мощность метрики «среднее количество просмотренных карточек товаров» в зависимости от длительности эксперимента

#### Влияние окружения на мощность

Мощность зависит не только от выбора метрики, но и от изменения значения метрики, которое можно получить в А/Б-тесте. Если изменение будет мало, то вероятность зафиксировать значимый результат тоже будет мала. Поэтому важно организовать эксперимент таким образом, чтобы влияние новой функциональности на посетителей страницы, а, следовательно, влияние на метрику было достаточно сильным.

Сила этого влияния зависит не только от того, что тестируется, но и от того, что уже находится на странице. Например, если проверяется влияние блока рекомендаций с новым алгоритмом «Похожие товары», то важно, чтобы на странице не было другого инструмента, выполняющего ту же задачу. Иначе часть посетителей продолжит пользоваться старым инструментом и не заметит нового, даже если новый инструмент показывает более качественные рекомендации. В таком случае в сегменте с тестируемым алгоритмом лучше убрать старый инструмент.

Кроме того, магазины часто недооценивают влияние расположения блоков рекомендаций на странице. Но от этого напрямую зависит то, насколько сильно изменится поведение пользователей. Например, блок рекомендаций, расположенный слишком низко, не окажет большого влияния вне зависимости от качества рекомендаций.

Частой ошибкой является использование слишком общих или не соответствующих алгоритму рекомендаций заголовков. Они должны отражать задачу, которую помогает решить пользователю алгоритм рекомендаций. Неподходящий заголовок может ввести пользователя в заблуждение, и в конечном итоге он перестанет обращать внимание на рекомендации. Иными словами, не будет обеспечиваться [свойство логичности](https://habr.com/ru/companies/retailrocket/articles/782510/#:~:text=%D0%9B%D0%BE%D0%B3%D0%B8%D1%87%D0%BD%D0%BE%D1%81%D1%82%D1%8C%20%E2%80%94%20%D0%BF%D0%BE%D0%BD%D1%8F%D1%82%D0%BD%D0%BE%2C%20%D0%BA%D0%B0%D0%BA%D1%83%D1%8E%20%D0%B7%D0%B0%D0%B4%D0%B0%D1%87%D1%83%20%D0%BF%D0%BE%D0%BA%D1%83%D0%BF%D0%B0%D1%82%D0%B5%D0%BB%D1%8F%20%D1%80%D0%B5%D1%88%D0%B0%D0%B5%D1%82%20%D0%B0%D0%BB%D0%B3%D0%BE%D1%80%D0%B8%D1%82%D0%BC).

Не менее важным является то, насколько качественно интегрирована система рекомендаций на сайте интернет-магазина. В качество интеграции входят такие компоненты, как передача актуальных данных по товарам и взаимодействиям пользователей с ними, вёрстка и правильный вызов элементов системы. Всё это может сказаться на влиянии тестируемой функциональности, и, как следствие, на мощности метрик.

### 3. Недостаточное понимание того, на что влияет функциональность

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/344/a30/501/344a30501bb1f081f692048d3390dd44.png)

При проведении А/Б-тестов важно понимать, на что влияет тестируемая функциональность. От этого зависит выбор метрик, которые используются при анализе результатов. От результатов, в свою очередь, зависит, какое решение будет принято.

#### Рекомендации могут влиять на конверсию в покупателя

Одной из метрик, которую мы отслеживаем в А/Б-тестах рекомендаций является «конверсия в покупателя». Она рассчитывается, как отношение количества покупателей к количеству всех пользователей. Влияние рекомендаций на эту метрику зачастую остаётся без внимания. 

В то же время наш опыт проведения А/Б-тестов говорит о том, что рекомендации способны влиять на конверсию в покупателя. Более того, это может происходить на разных страницах интернет-магазина, начиная с главной страницы, на которую заходит много новых пользователей, заканчивая страницей корзины, на которую в основном заходят пользователи перед оформлением заказа.

Увеличение конверсии в А/Б-тесте означает в конечном счёте положительное влияние на выручку интернет-магазина. Это связано с тем, что:

- Мощность конверсии выше, чем мощность метрик, связанных с выручкой. Соответственно выше вероятность того, что значимый результат по конверсии проявится в А/Б-тесте раньше, чем результат по другим метрикам;
    
- Польза в денежном эквиваленте от прироста количества покупателей может быть не сразу заметна из-за краткосрочности А/Б-тестов. В [нашем исследовании](https://habr.com/ru/companies/retailrocket/articles/595469/) мы показали, что увеличение активности пользователей, в том числе такой как совершение заказов, принесёт дополнительную прибыль магазину в долгосрочной перспективе. 
    

Далее мы расскажем, к чему может привести непонимание того, что рекомендации влияют на конверсию в покупателя.

Для измерения ценности продукта некоторые магазины используют метрику «средняя выручка на платящего пользователя» — ARPPU. Отличие этой метрики от средней выручки на пользователя в том, что она считается не по всем пользователям, а только по тем, кто сделал хотя бы один заказ. По сути она должна отражать реакцию платящих пользователей на продукт. Но ошибкой является попытка измерить ARPPU в А/Б-тесте рекомендаций. 

Для иллюстрации проблемы приведём пример А/Б-теста с двумя сегментами А и Б:

1. В сегменте А шесть посетителей, среди которых трое совершили покупки на 10р каждый. ARPPU равен 30/3=10р, средняя выручка на пользователя 30/6=5р;
    
2. В сегменте Б из шести посетителей трое совершили покупки на 10р каждый. Помимо этого рекомендации помогли трём остальным посетителям сделать покупки по 1р. ARPPU равен 33/6=5.5р, средняя выручка на пользователя 33/6=5.5р.
    

Видно, что в сегменте Б больше покупателей и больше выручка. Более того, новые три покупателя в дальнейшем могут продолжить делать покупки, что приведёт к увеличению LTV. Но значение метрики ARPPU не отражает положительного влияния рекомендаций, и даже наоборот – свидетельствует об обратном.

Дело в том, что тестируемая функциональность не должна влиять на выбор аудитории, по которой считается метрика ([Lesson 6](https://www.kdd.org/kdd2016/papers/files/adf0853-dengA.pdf)). Как в примере выше, наличие рекомендаций влияет на количество покупателей, которые являются аудиторией для метрики ARPPU. Если же предполагается использование подобных метрик, то нужно учитывать только данные, которые были доступны до начала эксперимента. В случае ARPPU необходимо брать в расчёт только тех пользователей, у которых были заказы до начала А/Б-теста.

#### Влияние на разные группы пользователей может отличаться

Различные группы посетителей могут по-разному взаимодействовать с интернет-магазином. Поэтому отличаться может и влияние новой функциональности на них. А значит, можно лучше понять пользу от тестируемой функциональности, если смотреть на результаты эксперимента по этим группам отдельно.

Примером таких групп являются новые и старые пользователи. Новые посетители — это те, кто впервые зашли на сайт во время эксперимента, старые — те, кто уже заходили до его начала. Разные рекомендации могут по-разному влиять на эти группы. Новые пользователи ещё не знакомы с ассортиментом магазина и могут быстро уйти, если не найдут товар, который их заинтересует. Поэтому в рекомендациях им важно показать популярные товары магазина, которые способны удержать и одновременно познакомить с ассортиментом магазина. В свою очередь, для старых пользователей известно предыдущее поведение и интересы. Им важно показать персональную подборку товаров, которые соотносятся с ранее решаемыми задачами.

Если в эксперименте мы видим результат по одной из таких групп пользователей, но не видим результата по всем пользователям в совокупности, то это не значит, что пользы от новой функциональности нет. Пользователи оставшихся групп с большой вероятностью будут занижать совокупную разницу значений метрики между сегментами А/Б-теста, тем самым занижая мощность. Поэтому при тестировании рекомендаций советуем смотреть ещё и на то, как меняются метрики, например, по новым и старым пользователям. И если есть значимые результаты, следует принимать их во внимание.

Кроме того, влияние на различные группы пользователей может лучше отражаться разными метриками. Так, по новым пользователям может чаще меняться конверсия, в то время как по старым — средний чек или средняя выручка на пользователя.

Другой пример групп пользователей возникает, когда функциональность затрагивает несколько страниц. В этом случае может возникнуть вопрос о том, как функциональность отдельной страницы влияет на её посетителей. Для этого необходимо получить результаты только по группе пользователей, которые заходили на эту страницу. 

Если эксперимент проводится одновременно на нескольких страницах, то возникнет проблема, о которой мы писали ранее — тестируемая функциональность повлияет на выбор аудитории. Например, если тест проводится одновременно на страницах карточки и корзины, функциональность карточки будет влиять на то, попадёт ли пользователь на страницу корзины. Таким образом, не получится запустить один эксперимент на нескольких страницах сразу и выделить аудиторию для отдельных страниц.

Поэтому для каждой страницы магазина мы организуем отдельный эксперимент. Кроме того, независимое деление трафика на каждой странице позволяет нам запускать несколько таких экспериментов одновременно.

### 4. Ожидание результата от единичного эксперимента

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/7fd/7d6/f37/7fd7d6f378fe90f3485cfc552a55afd0.png)

Какой бы полезной не была новая функциональность, сложно обнаружить эту пользу в рамках одного единственного эксперимента. Часто для этого необходимо организовать серию из множества А/Б-тестов. Как это сделать – зависит от того, в рамках какого процесса проводится эксперимент. Ниже мы опишем примеры таких процессов, которые встречаются в рекомендациях.

#### Тестирование одной функциональности для многих клиентов

При разработке нового алгоритма или улучшении существующего алгоритма рекомендаций необходимо подтвердить пользу от новой функциональности. Для этого в обязательном порядке нами проводятся А/Б-тесты на множестве различных магазинов. Сначала мы запускаем несколько А/Б-тестов на небольшом множестве магазинов и как можно быстрее проверяем отсутствие сильных изменений метрик вследствие возможных ошибок. Если таковых не обнаруживается, мы проводим большое количество экспериментов с целью окончательно определить, успешно ли выполняются поставленные задачи. 

На одном магазине в одном А/Б-тесте не получится достоверно определить, действительно ли новая функциональность приносит пользу. Могут возникнуть проблемы, часть которых была описана выше. Но множество А/Б-тестов на множестве магазинов позволяет нам получать достоверные результаты.

#### Поиск оптимальной конфигурации для одного клиента

Иногда возникает потребность найти оптимальную конфигурацию рекомендаций. Конфигурация определяет количество, расположение блоков рекомендаций на странице магазина, используемые алгоритмы рекомендаций и всё, что с этим связано. В рамках этого процесса для каждой страницы магазина:

1. Из прошлого опыта выбираются подходящие варианты конфигураций;
    
2. Проводится А/Б-тест;
    
3. Определяется наилучшая конфигурация по результатам А/Б-теста;
    
4. Выбираются новые варианты конфигураций, к ним добавляется конфигурация из пункта 3 и снова выполняется пункт 2.
    

Получить наилучшую конфигурацию за одну итерацию А/Б-тестов не представляется возможным, как и в случае выше. Но за достаточно большое количество экспериментов нам удаётся найти оптимальную конфигурацию для каждой страницы магазина. Она, в свою очередь, позволяет клиенту получать наибольшую эффективность от рекомендаций.

#### Тестирование эффективности продукта для одного клиента

В некоторых случаях клиент считает важным провести А/Б-тест с целью выяснить, какую прибыль приносит продукт. Для этого в одном из сегментов теста продукт не показывается пользователям. После завершения эксперимента рассчитывается разница средней выручки на пользователя между сегментами с наличием и отсутствием продукта. На основании чего производится расчёт общей выручки.

При проведении подобных экспериментов магазины часто проводят один А/Б-тест и останавливаются. Но в одном отдельном эксперименте велика вероятность возникновения проблем, которые сделают его результаты недостоверными. Часть таких проблем мы описали выше. Поэтому каждый эксперимент требует исследования их наличия. В случае обнаружения проблем необходимо их исправить и запустить очередной А/Б-тест. Таким образом, данный вариант тестирования предполагает больше одного эксперимента, как и тестирование любой другой функциональности магазина. 

При этом в таких экспериментах учитывается изменение выручки только во время теста. Но если тестируемый продукт будет работать дольше, то он может повлиять по-другому. Так, например, если рекомендации увеличивают количество добавлений в корзину, выручка не обязательно увеличится сразу. Но через какое-то время пользователи начнут заказывать те товары, которые ранее добавили в корзину, и долгосрочная выручка увеличится. Выше мы уже писали о том, что разные метрики в разной степени отражают влияние на долгосрочную выручку.

Кроме того, существуют метрики, которые могут показать ценность продукта без проведения А/Б тестов. Про такие метрики, а также про другие способы решения задач, связанных с оценкой влияния новой функциональности и продукта в целом на пользователей, мы напишем далее.

### О многообразии способов решения задач бизнеса, для которых принято использовать А/Б-тесты

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/2df/ddf/5e3/2dfddf5e3d20abc871ad89bb66531d64.png)

При проведении А/Б-тестов существует множество сложных аспектов. Особенно много их возникает при тестировании эффективности продукта. И только если учесть каждый из этих аспектов, можно рассчитывать на достоверный результат. В противном случае появится много недостоверных результатов. Бесполезные и даже вредные для магазина результаты будут использованы. При этом создастся видимость успеха, но в конечном итоге бизнес будет терять деньги.

Учёт всех аспектов требует значительных усилий многих специалистов, таких как аналитики, разработчики, маркетологи, и занимает продолжительное время. Кроме того, во время некоторых А/Б-тестов такой полезной функциональности, как система рекомендаций, не будет на части трафика. Всё это является потенциальной потерей денег для бизнеса. 

Чтобы снизить перечисленные выше издержки, можно предложить следующие варианты:

- Для уменьшения времени тестирования и увеличения вероятности получения достоверного результата можно использовать не только выручку, но и более чувствительные и менее шумные метрики. Про это мы писали в одной из своих [предыдущих статей](https://habr.com/ru/companies/retailrocket/articles/646455/).
    
- При тестировании рекомендаций лучше использовать более простой процесс А/Б-тестирования — тесты на поиск оптимальной конфигурации. В этом случае рекомендации будут видны всем пользователям и потеря денег от их отсутствия не будет такой существенной. Поэтому можно проводить множество таких тестов и итерационно находить всё более эффективные варианты. В то же время нахождение более удачного варианта в таком тесте свидетельствует о большей пользе от рекомендаций.
    
- Вместо сравнения метрик между сегментами в А/Б-тестах можно использовать метрики, значения которых свидетельствуют о ценности продукта сами по себе. Например, метрики, связанные с тем, как пользователи взаимодействуют с продуктом. Так, если достаточно большая доля товаров в заказах была найдена с помощью системы рекомендаций, то система обладает ценностью. Такой подход с атрибуцией широко используется в маркетинге. 
    
- Качество продукта можно оценить по наблюдаемым свойствам. В нашем блоге недавно вышел [цикл статей](https://habr.com/ru/companies/retailrocket/articles/782510/) о свойствах, которыми должны обладать качественные рекомендации, и о том, как проверить, что они действительно обладают этими свойствами. Так, например, качество рекомендаций можно оценить с помощью свойства логичности. Усиление данного свойства означает, что после изменения рекомендации будут лучше соответствовать ожиданиям пользователей магазина, а значит, станут более полезными. Тогда пользователи будут с большей вероятностью совершать покупки и возвращаться за следующими. Улучшение пользовательского опыта, в свою очередь, делает покупателей более лояльными магазину. Всё это ведёт к увеличению прибыли, которую получает бизнес за всё время работы с клиентами.