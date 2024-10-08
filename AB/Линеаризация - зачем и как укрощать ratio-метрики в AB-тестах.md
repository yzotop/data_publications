---
tags:
  - data
data_type:
  - AB tests
author: Mosin
link: https://habr.com/ru/companies/sbermarket/articles/768826/
source: habr
company: Kuper
---
[[База - айсберг AB тестов]]
[[Бутстрап швейцарский нож аналитика в AB-тестах]]


## References

1. Оригинальная статья — [Consistent Transformation of Ratio Metrics for Efficient Online Controlled Experiments](https://www.researchgate.net/publication/322969314_Consistent_Transformation_of_Ratio_Metrics_for_Efficient_Online_Controlled_Experiments);
2. Илья Кацев — [Как измерить счастье пользователя](https://www.youtube.com/watch?v=vIdwgJFz5Mk);
3. Роман Будылин — [Как делать чувствительные метрики для АБ-тестирования](https://www.youtube.com/watch?v=PE29k3CP72U);
4. Виталий Полшков — [Эффективное А/Б тестирование](https://www.youtube.com/watch?v=4sG40hyQ7WI).


## Введение

Привет, Хабр! В [прошлой статье](https://habr.com/ru/companies/sbermarket/articles/774608/) я указал, что в A/B-тестах используются три основных типа метрик, а именно **пользовательские** **конверсии**, **средние метрики пользователей** и **ratio-метрики**. К последним обычно относят средний чек, CTR баннера, среднюю длину сессии и др. Такие метрики имеют ограничения при оценке стандартными статистическими критериями и общую особенность определения в контексте экспериментов.

В этой статье формализуем понятие ratio-метрики, подробнее и на примере посмотрим на их ограничения и разберем, как инвалидировать результаты своих экспериментов, если эти ограничения игнорировать. Откроем для себя метод линеаризации ratio-метрик, разберем, как и почему он работает, какая интерпретация стоит за его преобразованием, а также определим его преимущества в сравнении с предусредненным средним, бутстрапом и дельта-методом.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/7ec/d22/433/7ecd22433bbe972d1697a577a8b09c56.png)

❗**Основная идея**❗

Линеаризация позволяет перейти от ratio-метрики с зависимыми наблюдениями к средней пользовательской метрике с независимыми наблюдениями. При этом:

- В экспериментах разница линеаризованных метрик сохраняет **сонаправленность** наблюдаемого изменения с изменением в целевой ratio-метрике;
    
- Уровень статзначимости наблюдаемого эффекта новой метрики **консистентен** уровню статзначимости исходной ratio-метрики и рассчитывается t-тестом;
    
- К линеаризованной метрике можно применять методы повышения чувствительности, такие как CUPED или стратификация.
    

Маленькое напоминание про термины

---

## 1. Формально про ratio-метрики

Введем два термина, а именно **единица анализа** и **единица рандомизации**.

**Единица анализа** — это сущность относительно которой мы хотим посчитать метрику. Ей может быть пользователь, сессия, заказ, кнопка, баннер, временной промежуток и тп.

К примеру, есть общий объем денег, полученный за какой-то период. Если мы его поделим (нормируем) на всех пользователей, то получим ARPU, а если на заказы — средний чек.

Также, можно сгруппировать данные по пользователям и для каждого определить его траты, построить их распределение, а средним значением этого распределения и будет ARPU. Проделав то же самое с заказами, в распределении их цен средним значением будет средний чек. Мы просто перераспределяем общий массив чего-то на какие-то различные единицы анализа.

![Распределение массива денег по разным единицам анализа](https://habrastorage.org/r/w1560/getpro/habr/upload_files/7c4/05d/a87/7c405da8719433046cdbb306f41b158c.png "Распределение массива денег по разным единицам анализа")

Распределение массива денег по разным единицам анализа

**Единица рандомизации** — это сущность, которую мы случайным образом назначаем в тестовую или контрольную группы в А/B-тестах, ей также может быть пользователь, сессия, заказ или временной промежуток. Но на практике чаще всего рандомизируют именно пользователей, на их примере и будем разбираться дальше.

Рандомизация помогает нивелировать влияние ненаблюдаемых факторов, которые могут исказить результаты эксперимента. К этим факторам можно отнести любой номинативный признак пользователя, например, возраст, пол, геолокацию, а также любой другой с менее четкой формулировкой — «излишне активный», «прокликивает только выгодные акции», «неуверенный/не знает что выбрать» и тп.

Благодаря рандомизации мы знаем, что наблюдаемая разница между значениями метрик в группах возникла в результате одного из двух вариантов:

- Либо из-за нашей экспериментальной механики;
- По чистой случайности, т.е. рандомное назначение в группы, возможно, привело к тому, что более результативные пользователи оказались в одной группе.

Кроме того, рандомизация обеспечивает **независимость** полученных от пользователей сигналов. Так, траты или клики одного пользователя не зависят от действий другого. Независимость наблюдений — одно из главных свойств для применения статистического критерия к данным.

Когда в эксперименте совпадает единица анализа и единица рандомизации, то смело можно применять статкритерий для проверки гипотезы. Это относится к любой пользовательской конверсии и средней пользовательской метрике, поскольку для одной метрики мы имеем только одну выборку независимых пользовательских сигналов.

![Применение статтеста к разным сигналам единицы рандомизации (пользователя)](https://habrastorage.org/r/w1560/getpro/habr/upload_files/81e/98f/918/81e98f91816ae05c2c3311f1a995a5e8.png "Применение статтеста к разным сигналам единицы рандомизации (пользователя)")

Применение статтеста к разным сигналам единицы рандомизации (пользователя)

Из-за требования статкритериев к независимости наблюдений все сигналы в A/B-тестах справедливо определять только в разрезе единицы рандомизации, т.е. пользователя.

Когда в эксперименте хочется посчитать метрику относительно другой единицы анализа, отличной от единицы рандомизации, то появляется **ratio-метрика**, которая имеет особенность в формальном определении через единицу рандомизации.  
Интересующая в эксперименте единица анализа всегда находится в **зависимости** от единицы рандомизации. При этом сами зависимые сущности и их сигналы можно «свернуть» до двух соответствующих пользовательских сигналов. К примеру, если у пользователя есть три заказа с определенными ценами, то для этого пользователя нужно просто посчитать число заказов и суммарные траты по ним. Проделав так для всех пользователей, дальше можно сложить сигналы по двум новым полям и поделить их суммы друг на друга, так получится среднее значение для интересующей единицы анализа, в нашем случае — средний чек.

**Ratio-метрика** — это метрика отношения непользовательского уровня (non-user level metric) с зависимыми наблюдениям, но которая явным образом выражается через отношение сумм соответствующих пользовательских сигналов.

![Формальное определение среднего чека через единицу рандомизации. Ratio-метрика](https://habrastorage.org/r/w1560/getpro/habr/upload_files/c69/104/099/c691040996b02d29f811dad61cf8ce97.png "Формальное определение среднего чека через единицу рандомизации. Ratio-метрика")

Формальное определение среднего чека через единицу рандомизации. Ratio-метрика

## 2. Почему нельзя считать ratio-метрики t-тестом?

Формально — из-за нарушения предпосылок о независимости наблюдений, поскольку цены заказов, длины сессий, конверсии показов в клик и тп. в рамках одного пользователя адекватно считать _скоррелированными_. Получается, что принадлежащие одному пользователю единицы анализа и значения их сигналов **зависят** от субъективных характеристик этого самого пользователя.

Неформально это тезис можно проверить на синтетических A/A-тестах, распределяя по разным группам пользователей, а статзначимость оценивая, например, по их заказам. При этом распределение _p-value_ для ratio-метрики будет отличаться от равномерного. В свою очередь, распределение _p-value_ для средней пользовательской метрики по таким же сигналам будет иметь корректное равномерное распределение.

Код на Python

```

```

![Сравнение А/А-тестов на одинаковых сигналах для ratio-метрики и средней пользовательской](https://habrastorage.org/r/w1560/getpro/habr/upload_files/cc0/ee2/0df/cc0ee20dfd858575ef69d270a964ab03.png "Сравнение А/А-тестов на одинаковых сигналах для ratio-метрики и средней пользовательской")

Сравнение А/А-тестов на одинаковых сигналах для ratio-метрики и средней пользовательской

Получается, если расчитывать ratio-метрики t-тестом, то в ваших экспериментах будет расти ошибка 1-го рода, что приведет к значительному росту добавленных в продукт механик-пустышек без положительного влияния на бизнес.

Тогда встает логичный вопрос, а как в таком случае оценивать на статзначимость ratio-метрики, и здесь есть 3 распространенных варианта:

1. Посчитать через прокси-метрику, выраженную предусредненным средним значением на пользователя.
    
2. С помощью бутстрапа.
    
3. С помощью дельта-метода.
    

И каждый из них имеет свои недостатки. Давайте разбираться на примере среднего чека, CTR баннера и средней длины сессии.

## 3. Методы статистической оценки разницы ratio-метрик и их минусы

### 3.1. Предусредненное среднее

Наивное решение заключается в том, чтобы преобразовать ratio-метрику к средней пользовательской прокси-метрике, предварительно усреднив по пользователю соответствующие сигналы. Таким образом получится предусредненное среднее пользовательских значений. На картинке ниже указана общая формула для среднего чека на пользователя, CTR на пользователя и средней длины сессии на пользователя.

![Определение предусредненных пользовательских сигналов](https://habrastorage.org/r/w1560/getpro/habr/upload_files/04a/4f1/d83/04a4f1d8311cab2c235d49db0ac21ace.png "Определение предусредненных пользовательских сигналов")

Определение предусредненных пользовательских сигналов

Для такой метрики корректно считать значимость стандартными статтестами, так как эти сигналы независимы. Однако у такой метрики есть проблема — в экспериментах наблюдаемый эффект в предусредненном среднем и в целевой ratio-метрике могут иметь разный знак, т.е. эффекты могут быть **разнонаправлены**. Например, средний чек на пользователя может статзначимо вырасти, но реальный глобальный средний чек мог упасть. Проверить это можно либо на корпусе проведенных экспериментов, либо с помощью симуляций.

![Сравнение сонаправленности эффектов в предусредненном среднем и в целевой ratio-метрике](https://habrastorage.org/r/w1560/getpro/habr/upload_files/cd5/21b/d79/cd521bd7933751b4cdf38157f4eb46ae.png "Сравнение сонаправленности эффектов в предусредненном среднем и в целевой ratio-метрике")

Сравнение сонаправленности эффектов в предусредненном среднем и в целевой ratio-метрике

На картинке по оси _Ox_ наблюдаемые изменения в предусредненном среднем, по _Oy_ эффект в ratio-метрике. Сонаправленные изменения выделены зелеными точками, а разнонаправленные — красными. Получается из-за того, что в экспериментах может отсутствовать сонаправленность эффектов между этими метриками, то предусредненное среднее является плохой прокси-метрикой для замера статзначимости в ratio-метрике.

### 3.2. Бутстрап

Когда не знаешь как посчитать статзначимость для своей метрики — [бутстрап](https://habr.com/ru/articles/762648/) в помощь.

Фактически проблема статоценки разницы ratio-метрик t-тестом связана с зависимостью наблюдений. Нельзя просто взять и по группам посчитать выборочную дисперсию и оценить стандартную ошибку среднего (_standard error_) для разницы ratio-метрик. А вот с помощью бутстрапа можно.

Для ratio-метрики надо семплировать с повторением случайных пользователей в группах и отбирать их соответствующие сигналы для числителя и знаменателя, считать разницу бутстрапированных ratio-метрик и повторять так много-много раз. По итогу получится эмпирическое распределение разницы ratio-метрик, среднем значением которого будет наблюдаемый эффект в эксперименте, с какой-то оценкой ее вариативности, эмпирической стандартной ошибкой среднего. Из этого распределения уже можно посчитать заветный _p-value._

Как бутстапить ratio-метрику на Python

```

```

![Эмпирические распределения разниц ratio-метрик, полученные бутстрапом](https://habrastorage.org/r/w1560/getpro/habr/upload_files/6c3/629/251/6c3629251eb9c6ff1dc02eeb8c297c5f.png "Эмпирические распределения разниц ratio-метрик, полученные бутстрапом")

Эмпирические распределения разниц ratio-метрик, полученные бутстрапом

Бутстрап выдает корректные _p-value_ для ratio-метрик, но минус подхода очевиден — бутстрап вычислительно затратен и не масштабируется в рамках платформы экспериментов, где могут считаться десятки ratio-метрик.

### 3.4. Дельта-метод

По своей сути дельта-метод делает тоже самое что и бустрап, но уже не эмпирически, а аналитически, через формулу. С помощью этой формулы можно корректно пересчитать дисперсию для ratio-метрик в тесте и контроле, а зная дисперсии и число пользователей в группах, уже можно оценить стандартную ошибку среднего для разницы ratio-метрик и соответственно t-статистику с _p-value._ Подробнее про дельта-метод в [оригинальной статье](https://arxiv.org/pdf/1803.06336.pdf) или у [коллег](https://habr.com/ru/companies/X5Tech/articles/740476/) по цеху.

Дельта-метод на Python

```

```

![Суть дельта-метода](https://habrastorage.org/r/w1560/getpro/habr/upload_files/a54/dfd/a24/a54dfda2477271e7b0443d565316866b.png "Суть дельта-метода")

Суть дельта-метода

Если считать статзначимость бутстрапом и дельта-методом на одинаковых выборках для разницы ratio-метрик, то _p-value_ двух методов будут совпадать с достаточной точностью и иметь линейную зависимость с единичным угловым коэффициентом. Можно сказать, что значения _p-value_ дельта-метода **консистентны** значениям, полученным с помощью бутстрапа. Соответственно, и на A/A-тестах дельта-метод дает равномерные распределения для ratio-метрик и не завышает ошибку 1-го рода.

![Консистеность p-value дельта-метода и бутстрапа. Результататы A/A-тестов дельта-метода](https://habrastorage.org/r/w1560/getpro/habr/upload_files/b9e/43d/a73/b9e43da73881d98a9bcdf712866e4fcb.png "Консистеность p-value дельта-метода и бутстрапа. Результататы A/A-тестов дельта-метода")

Консистеность _p-value_ дельта-метода и бутстрапа. Результататы A/A-тестов дельта-метода

Недостатки у подхода тоже есть. Так как мы не работаем напрямую с пользовательскими сигналами, то для ratio-метрик не получится использовать методы повышения чувствительности, т.е. в экспериментах для пользовательских конверсий и средних пользовательских метрик можно применить CUPED, для ratio-метрик — нет.

Но есть один отличный метод, который лишен всех описанных выше недостатков.

## 4. Линеаризация

Определим функцию линеаризации двух пользовательских сигналов, чтобы для каждого пользователя получить один сигнал. Из этих новых сигналов соберем новую среднюю пользовательскую линеаризованную метрику, которая будет прокси к исходной ratio-метрике.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/03d/16d/2aa/03d16d2aa8fa2e673c368bff73736063.png)

Варианты на Python

```

```

```

```

А что за значения линеаризованных сигналов получились, как их воспринимать и как инерпритировать новую метрику?

К линеаризованным сигналам можно относиться как к единицам **вклада** в изменение ratio-метрики от начального значения в контроле до наблюдаемого в тесте. Этот тезис можно наглядно продемонстрировать на распределениях этих вкладов в экспериментальных группах.

Математическое ожидание линеаризованной метрики в контрольной группе всегда равняется нулю, т.е. суммарно в контроле вклада нет, так как относительно ratio-метрики в контроле и замеряем. В тестовой же группе будет какое-то другое среднее значение вкладов, либо положительное, либо отрицательное.

![Распределения линеаризованных пользовательских сигналов для разных метрик](https://habrastorage.org/r/w1560/getpro/habr/upload_files/7f2/5ee/e76/7f25eee760a338f081ba81289eabf81e.png "Распределения линеаризованных пользовательских сигналов для разных метрик")

Распределения линеаризованных пользовательских сигналов для разных метрик

При этом линеаризованная метрика обладает рядом положительных особенностей.

Во первых, в отличии от предусредненного среднего, разница линеаризованных метрик всегда сохраняет **сонаправленность** с изменением в целевой ratio-метрике. Например, если в эксперименте CTR вырос или упал, то линеаризованный CTR всегда изменится в ту же сторону.

![Сравнение сонаправленности эффектов с целевой ratio-метрикой для линеаризации и предусредненного среднего](https://habrastorage.org/r/w1560/getpro/habr/upload_files/fd1/3e0/e7a/fd13e0e7a8bbd2bee754e6caf8948296.png "Сравнение сонаправленности эффектов с целевой ratio-метрикой для линеаризации и предусредненного среднего")

Сравнение сонаправленности эффектов с целевой ratio-метрикой для линеаризации и предусредненного среднего

Во вторых, линеаризованные пользовательские сигналы уже можно считать независимыми и для них определять статзначимость t-тестом. При этом, значения _p-value_ для линеаризованной метрики будут **консистентны** значениям, полученным с помощью дельта-метода на исходной ratio-метрике. Это значит, что дельта-метод можно заменить линеаризацией и получать одинаковые значения _p-value_ с достаточной точностью. Соответственно и на А/А-тестах линеаризация показывает корректные результаты.

![Консистеность p-value линеаризации и дельта-метода. Результататы A/A-тестов линеаризации](https://habrastorage.org/r/w1560/getpro/habr/upload_files/10d/066/cf8/10d066cf8c5644f0591316ab10f571e8.png "Консистеность p-value линеаризации и дельта-метода. Результататы A/A-тестов линеаризации")

Консистеность _p-value_ линеаризации и дельта-метода. Результататы A/A-тестов линеаризации

В третьих, линеаризация позволяет использовать методы повышения чувствительности на ratio-метриках, чтобы уменьшать размеры экспериментальных групп для обнаружения эффектов или чтобы увеличивать мощность в наблюдаемых результатах. Например, чтобы использовать CUPED для ratio-метрики, необходимо ее линеаризовать и линеаризовать соответствующие сигналы на предэкспериментальном периоде. Получатся две средние пользовательские метрики, к которым уже можно применить CUPED.

На картинке внизу видно, что такой критерий оказывается более мощным в сравнении с обычной линеаризацией и дельта-методом.

![Сравнение мощностей для линеаризации, дельта-метода и линеаризации+CUPED](https://habrastorage.org/r/w1560/getpro/habr/upload_files/823/fba/45a/823fba45ad4269b78188e9b2bfc63b61.png "Сравнение мощностей для линеаризации, дельта-метода и линеаризации+CUPED")

Сравнение мощностей для линеаризации, дельта-метода и линеаризации+CUPED

## Вывод

Линеаризация в A/B-тестах легко вычисляемый и отлично масштабируемый метод трансформации ratio-метрики к средней пользовательской метрике. Она сохраняет **сонаправленность** наблюдаемого эффекта с изменением в целевой ratio-метрике. Также в экспериментах разница линеаризованных метрик имеет **консистентный** уровень статзначимости с исходной ratio-метрикой и рассчитывается t-тестом. Поскольку благодаря линеаризации мы получаем сигналы пользовательского уровня, то открываются возможности применять для ratio-метрики методы повышения чувствительности.

И именно линеаризация ratio-метрик с применением CUPED внедрена в нашу платформу A/B-тестов.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/cfb/2e0/2fa/cfb2e02fae638c370a3775a77bea6ca7.jpg)