---
tags:
  - data
source: medium
data_type:
  - AB tests
link: https://medium.com/statistics-experiments/%D0%BA%D0%BE%D0%B3%D0%B4%D0%B0-%D0%BE%D1%81%D1%82%D0%B0%D0%BD%D0%B0%D0%B2%D0%BB%D0%B8%D0%B2%D0%B0%D1%82%D1%8C-a-b-%D1%82%D0%B5%D1%81%D1%82-%D1%87%D0%B0%D1%81%D1%82%D1%8C-1-mde-7d39b668b488
author: expf
company: expf
---
[[Когда останавливать AB-тест. Часть 2 Monte Carlo]]


# Контекст и постановка задачи

Перед запуском эксперимента необходимо понять на какое время его запускать, чтобы принять решение о его эффективности. Это комплексный вопрос, затрагивающий целое множество факторов, оказывающие влияние на время. Грубо все их можно классифировать на две группы: статистические факторы и продуктовые.

![](https://miro.medium.com/v2/resize:fit:700/1*BjPn1Ral4nieTQFCZchRew.png)

Диграмма 1: Факторы, влияющие на время эксперимента

Под продуктовыми подразумевается, что на время влияет выбор метрики и сезонность продукта. У метрики тоже есть своя псевдосезонность, которую правильнее назвать “окном закрытия”. Под ним подразумевается, что пользователь может совершить какое-то действие не раньше времени t. Например, платеж для пролонгации в подписочном сервисе.

Также стоит обратить внимание на поведенческие пользовательские циклы, которые в свою очередь зависят от контекста целей использования сервиса и ее аудитории. Например, как долго принимается решение о первой покупке или как часто используется сервис.

У продукта может быть еще больше факторов, которые менее универсальны и не представлены в диаграмме. Тем не менее, учет приведенных составляющих из диаграммы уже позволяет определить некоторую безусловность.

С продуктовыми факторами все предельно ясно: есть индивидуальные особенности сервиса, которые мы не будем разбирать в рамках этого руководства, т.к. вариативность слишком широкая из-за бизнес-модели и отрасли присутствия .

Со статистикой же абстракции меньше, мы ее и рассмотрим.

# Что влияет на остановку теста?

## Подходы по определению оптимального момента остановки

Перед тем как разобрать статистические факторы, сделаем небольшое отступление и определим известные подходы в дизайне.

В статистике обычно разделяют две методологии по определению оптимального момента остановки эксперимента: Fixed time Horizon и Sequential testing.

## Fixed time Horizon

![](https://miro.medium.com/v2/resize:fit:700/1*3--sRHyFvfZ7tZ8EkhAuHg.png)

Процедуры Fixed horizon

В Fixed Horizon фиксируется временной горизонт проведения эксперимента, при котором один раз рассчитывают время, дожидаются и принимают решение о результатах теста. Для этого используют MDE (Minimum Detectable Effect).

Фиксированный дизайн более распространен в практике, т.к. он проще в понимании аналитиками и бизнесом и в определенном смысле универсален.

## Sequential testing

В Sequential testing принято смотреть на границы остановки эксперимента в реальном времени. Статистика накопительная и рассчитывается с помощью дополнительных порогов по уровню значимости, либо на основе апостериорной вероятности выигрыша тестового варианта.

![](https://miro.medium.com/v2/resize:fit:372/1*i6hKianDsVk7XMwjmAjwFA.jpeg)

Процедуры mSPRT. Источник [https://www.sciencedirect.com/science/article/pii/S0022249621000109](https://www.sciencedirect.com/science/article/pii/S0022249621000109)

По сути, процедуры последовательного тестирования — это семейство методов. Сюда входит [SPRT Вальда](https://en.wikipedia.org/wiki/Sequential_probability_ratio_test) (а именно его расширение — mSPRT) и [Байесовские бандиты](https://en.wikipedia.org/wiki/Multi-armed_bandit). Первое было популяризовано благодаря сервису optimizely.com. Второе инструментами Google Content Experiments, интегрированный в Google Analytics, который в будущем стал отдельным продуктом — Google Optimize.

У каждой методологии свои плюсы и минусы. Для более простой интеграции в экспериментальный пайплайн и понятной интерпретации для бизнеса больше подходит Fixed Horizon, но он проигрывает _проблемой подглядывания (peeking problem)._ Последовательное тестирование mSPRT привлекает как раз оптимизацией подглядываний, но имеет [сложный дизайн](https://www.sciencedirect.com/science/article/pii/S0022249621000109) и требователен к ресурсам поддержания разработкой. Бандиты также оптимизируют пикинг, но вовсе не является частотным подходом и распространен в узкой области применения (например, оптимизация показана офферов в витринах).

Процедуры последовательного тестирования было бы правильно рассмотреть в отдельных статьях, поэтому пока остановимся на Fixed Horizon. К тому же, задачу остановки теста лучше начинать с теории Fixed Horizon и затем двигаться в сторону изучения методов Sequential Testing

## Параметры

Вопрос, связанный со статистическими факторами универсален для всех типов бизнеса и продукта. Сюда стоит включить ожидания по размеру эффекта и выбор границ по ошибкам I и II типов. В диаграмме 1 также не с проста стоит отдельным пунктом дисперсия. Дисперсию можно сократить, благодаря техникам оптимизации. Например, степенные трансформации. Прежде, чем их рассматривать, лучше разобрать приведенные выше факторы, а затем уже начать их изучение.

Время эксперимента рассчитывается, исходя из необходимого объема выборки для обнаружения истинного эффекта, который станет известен уже по факту. Определимся с базовой терминологией и двинемся дальше

> _Ошибка второго рода (beta, False Negative)_ — принятие неверной нулевой гипотезы. Вероятность ошибочно сохранить неверную нулевую гипотезу обозначают греческой буквой β (например, 0.2)
> 
> _Ошибка первого рода (alpha, False positive)_ — отклонение верной нулевой гипотезы. Риск совершить такую ошибку равен выбранному 1-уровень статистической значимости и обозначают греческой буквой α (например, α=0.05)
> 
> _Мощность true positive)_ — вероятность, что тест правильно засечёт эффект там, где он и правда есть. (1-β)
> 
> _Статистическая значимость (True Negative)_ — вероятность, что результат эксперимента получился случайно и мы это знаем. Другими словами, это риск, что тест покажет взаимосвязь, которой на самом деле нет (1-α)

## Мощность и ошибка II рода

![](https://miro.medium.com/v2/resize:fit:700/1*YEBcANRvZGvFICzkwWV4EQ.png)

Мощность для продуктовых реалий является важным параметром, потому что никому не хотелось бы упускать эксперименты, действительно оказывающие влияние на продукт (это касается и отрицательных и положительных эффектов).

![](https://miro.medium.com/v2/resize:fit:700/1*EfpUIbPQhI2sb8Zj5g-rPg.png)

Диаграмма 2. Интерпретация мощности и ошибки II рода

Допустим, мы берем уровень мощности в 80% как минимальный допустимый порог. То из 1000 экспериментов с положительными изменениями в метрике, в 800 мы были бы уверенны, что изменение истинно существует. Остаются 200 тестов, которые будут ошибочно не признанными успешными, потому что остается False Negative (ошибка II) = 0.2. На диаграмме 2 пример только для тестов с положительным эффектом для упрощения понимания того, как работает мощность.

Мощность нужно максимизировать на столько, на сколько позволяют возможности продукта. В первую очередь это зависит от трафика (например, MAU). В целом, чем крупнее продукт по уровню выручки, тем сильнее надо занижать пороги рисков.

Порог 0.2 весьма вакуумный и не является универсальным. Какой порог приемлем определяют с помощью исследования чувствительности метрики. Как минимум один раз проделать это упражнение нужно, а дальше время от времени повторять для профилактики.

**Вывод:** Чем выше уровень мощности, тем меньше изменений в метрике будет упущено

## Уровень значимости и ошибка I рода

![](https://miro.medium.com/v2/resize:fit:700/1*Qf-83FEbsoiNOcI06hj8dQ.png)

Когда принимается решение о результатах эксперимента, считается, что в первую очередь нужно смотреть на p-value (ошибка I рода или false positive): положительный прокрас, отрицательный прокрас или просто — отсутствие разницы.

![](https://miro.medium.com/v2/resize:fit:700/1*HjIvD5nqW7nPYChncaXV-w.png)

Диаграмма 3. Интерпретация уровня значимости и ошибки I рода

Допустим, мы берем уровень значимости в 95% как минимальный допустимый порог для экспериментов. Значит из 1000 безуспешных экспериментов (нет ни отриц. ни полож. эффекта) в 950 мы были бы уверенны, что эффекта реально нет. Но 50 ошибочно выкатили бы в продакшен, хотя в этом нет никакого смысла, потому что в них мы наблюдаем случайное изменение метрики, а не реальную закономерность.

Так же как и мощность, уровень значимости необходимо увеличивать по мере возможностей. Классические 95% оторваны от современных реалий и требований бизнеса к точности. Следовательно, 5% кажутся не позволительной ошибкой для больших компаний. Есть отдельные случаи, когда порог по уровню значимости берут ниже. Это скорее исключение из правил, но при этом в голове всегда нужно держать мысль о возможных рисках.

**Вывод:** Чем выше уровень значимости, тем меньше бесполезных экспериментов будут выкатываться в продакшн.

## Минимальный ожидаемый эффект (MDE)

Хорошо бы иметь показатель, который будет учитывать пожелания по ошибкам обоих типов и к этому еще добавить ожидания по размеру эффекта.

Изменения, которые мы хотели бы увидеть с заданными порогами ошибок I и II типов рассчитываются с помощью _минимального ожидаемого эффекта_ (minimum detectable effect или MDE). Для этого параметра не хватит короткого описания. Разберем основные вопросы далее.

# Что такое MDE?

Начнем с того, что люди часто путают MDE и lift (или uplift). Типично, в обсуждениях имеют ввиду одно и то же, когда говорят про эффект, прирост, значимый прирост и прочие подобные. Но на самом деле это не так. В этих определениях семантика играет важную роль, когда заходит речь про оценку результатов A/B.

## Lift или прирост

Lift — это отличие метрики теста от метрики контроля. Считаем их разницу и делим на контроль. Умножаем на 100 и получаем %-ное изменение:

![](https://miro.medium.com/v2/resize:fit:700/1*9SFJD-JcySP7UuRn1WfXVQ.png)

Мы по факту на данных эксперимента рассчитываем прирост и говорим какая у метрик относительная разница. Тут пока ни о каких ошибках I и II типов речи не идет. Это обычный прокси параметр для оценки изменения метрики эксперимента.

Например, возьмем средний чек в контроле 1005 рублей, в тесте 1200. Тогда прирост = (1200–1005)/1005 = 0.194 = 19,4% .

## Effect Size

Effect Size (мера эффекта) — это стандартизированный прирост: считаем разницу и делим на дисперсию.

![](https://miro.medium.com/v2/resize:fit:700/1*Qmlo2xD4rsAivp3DImnBOA.png)

В формуле используется [pooled standard deviation](https://en.wikipedia.org/wiki/Pooled_variance). Как следует из формулы, для расчета знаменателя нам требуются либо данные по эксперименту, либо знания о метрике из исторических данных, иначе дисперсию не от куда взять.

Чем больше полученное число, тем сильнее размер эффекта. Это справедливо, если дисперсия устойчива и почти не изменяется. На практике мы знаем, что по мере увеличения охвата временного периода в выборке меняется и дисперсия. Поэтому нормировать величину нужно с мыслью о том, за каком период выборка была взята.

Effect size измеряет степень отклонения наблюдаемых значений от тех, которые можно было бы ожидать при отсутствии эффекта. Т.е., считая разницу к дисперсии можно понять насколько сильно отклонился наблюдаемый эффект от эффекта, который равен нулю. Концепции Effect Size и Power analysis были предложены [Jacob’ом Cohen’ом](https://en.wikipedia.org/wiki/Jacob_Cohen_(statistician)) и, если, какие-то моменты с Effect Size останутся не очевидными, имеет смысл обратиться к первоисточнику.

Effect size считается по разному в зависимости от типа метрики (= распределения), но смысл остается. Выше показана формула для [cohens’ d](https://en.wikipedia.org/wiki/Effect_size) для непрерывного распределения с гипотезой о разнице средних, что согласуется с t-тестом.

Например, средний чек в контроле 1005 рублей, в тесте 1200, SD_pooled = 150. Тогда Effect size = (1200–1005)/150 = 1.3. Это значит, что на 1.3 стд. эффекта отклонилась величина

**Интерпретация**

![](https://miro.medium.com/v2/resize:fit:414/1*8kSZ7UTYo1LJZcQENuJ_lg.png)

Таблица 1. Значения d и их интерпретация

Остается открытым вопрос –как проинтерпретировать полученное число.Узнали мы 1.3, что с этим делать? Cohen предлагал ориентироваться на референсную таблицу (см. таблица 1) для удобства. Однако, стоит понимать, что в больших абсолютных масштабах маленький значимый эффект может принести миллионы дополнительного дохода и поэтому ориентиры скорее условные и уже не имеют отношения к реальности

Более низкий cohen’s d указывает на необходимость более крупного размера выборки, и наоборот, что впоследствии может быть определено вместе с дополнительными параметрами желаемого уровня значимости и мощности.

**Вывод:** Effect size обладает большей информативностью и точностью с точки зрения оценки прироста, т.к. учитывает дисперсию и размер выборки.

## MDE

![](https://miro.medium.com/v2/resize:fit:700/1*cuOafp6P1EQXo30X-75LKA.png)

График 1: по вертикали — Мощность, по горизонтали — Объем выборки (N), α — ошибка I рода. В скобках условные обозначения силы эфекта

Минимальный ожидаемый эффект — это наименьший **истинный эффект** полученный от изменений, который с уверенностью сможет обнаружить статистический критерий.

Еще точнее: это наименьший истинный эффект полученный от изменений, который имеет **определенный уровень статистической мощности для определенного уровня статистической значимости**, учитывая конкретный **статистический тест**.

Разберем то, что написано выше:

- _…истинный эффект…_ — здесь имеется ввиду, что если прокраса (значимого изменения метрики) при `n = 10000` у такого эффекта нет (например, 2%), то достаточно уверенно можно сказать лишь то, что возможный истинный эффект не больше, чем MDE. Если выборка была бы больше, скажем `n = 20000` , то может эффект мы бы и увидели, но точно меньше (т.е. < 2%).
- _…определенный уровень статистической мощности для определенного уровня статистической значимости…_ — в расчетах MDE участвуют альфа, бета и размер выборки. Исходя из заданных уровней мощности и значимости, MDE будет варьироваться при одном и том же объеме выборки. Это можно понять на Графике 1: чем ниже альфа и бета, тем больше потребуется пользователей для одного и того же MDE, т.к. завысили требования к точности.
- _…статистический тест_ — в формуле расчета MDE учитывается распределение из которого взята метрика. MDE по разному рассчитывается для t-критерия, хи-квадрата и других статических критериев.

## Как принимать решение на основе MDE?

Если прокраса в метрике нет, то это не означает отсутствие эффекта. Эффект может и есть, но он точно не больше, чем MDE для α, β и дисперсии.

Например, вы наблюдаете 14-й день эксперимента, MDE на уровне 1%, значимого изменения нет. Это означает, что если вы продолжите эксперимент и увидите значимое изменение, то только для эффекта равный или меньший 1%. Принимать решение продолжать эксперимент или нет — можно, принимая в расчет MDE.

Эвристическая оценка того, что надо “ждать/не ждать” или того, какой эффект выбрать в качестве временной отсечки, рождается скорее из интуиции, чем из статистики. MDE здесь лишь выполняет роль мета-метрики или дополнительного индикатора эксперимента.

Почему вам вдруг захочется ждать эффект < 1% может быть обусловлено многим. Например, 1% эффекта может принести дополнительно 100000 заказов в месяц с некоторым доверительным интервалом. Если же этот 1% не имеет практической пользы для бизнеса, то смысла в ожиданиях конечно же нет. Отсюда следует и другая истина — много пользователей не бывает даже у продуктов с емким траффиком. На стадии стагнации роста, когда все низковисящие фрукты сорваны, а на реализацию смелых гипотез нет приоритетов, приходится очень долго ждать, чтобы тот самый < 1% эффекта прокрасился. Даже с миллионами пользователей

# Как считать MDE?

Выше мы начали разбирать Effect Size для t-теста, поэтому продолжим с ним и дальше. Достаточно на его примере разобрать логику, чтобы наследовать и для остальных критериев, т.к. смысл и порядок вычислений не изменится. Ниже показана формула вывода эффекта из дисперсии для непрерывного распределения (при неравных дисперсиях).

![](https://miro.medium.com/v2/resize:fit:700/1*8nf6KQROEeqkW01I--I3KA.png)

Для равных дисперсий ошибка немного изменится, но суть останется та же:

![](https://miro.medium.com/v2/resize:fit:700/1*mIOQCbkAHAjmhkE0waMapw.png)

В М считаем t-значения для параметров альфа и бета. Для двусторонней проверки альфу делим на 2. Далее когда получим MDE, мы будем уверены в результатах, согласно выбранным порогам значимости и мощности.

Если нас интересует уровень значимости 99%, то соответственно берем альфу 0.01 и получаем `t_alpha/2 = 2.58` Расчет для беты: `t_beta = 0.842`для 80% мощности. Тогда `M = 2.58 + 0.842 = 3.422` . Отношение к среднему по контролю `Y_c` здесь для получения процента (= lift).

Для расчета размера выборки можно “провернуть фарш обратно” и, подставив необходимый Effect Size, получить соответствующее значение n. Формула ниже работает для сбалансированного теста и вернет значения для одной выборки:

![](https://miro.medium.com/v2/resize:fit:700/1*-GwpAdrFHvQKQ4ZhuWVgUA.png)

Так же можно сделать с поиском неизвестной области мощности и альфы. Имея знания по Effect Size и размеру выборки, таким образом вычисляется альфа и бета:

![](https://miro.medium.com/v2/resize:fit:700/1*qgFyf_Izie-QpCQvpjmZhg.png)

Зная 3 из 4 членов, всегда есть возможность найти неизвестное значение пропущенного члена. Это удобно и гибко, т.к. могут всплывать вопросы рода:

- Сколько необходимо наблюдений для проведения эксперимента?
- На какой эффект я могу рассчитывать, если обладаю ограниченной базой пользователей?
- Насколько я могу доверять полученному эффекту по итогам теста?
- и т.п.

# Примеры на Python

Допустим, мы запустили A/B. Было накоплено по ~1004 наблюдений в каждой ветке эксперимента. Мы хотим проверить метрику с непрерывным распределением и узнать какой “точности” мы добились за это время. Для этого посмотрим на MDE и проинтерпретируем результаты:

Мы получили в качестве результата 0.125. Это означает, что эффекты ниже этого значения будут обладать меньшей мощностью. Соответственно, вероятность того, что эффект действительно есть будет меньше. Эффекты выше этого значения, наоборот, будут мощнее.

Теперь сделаем обратную операцию и рассчитаем количество наблюдений, зная Effect Size. При тех же значениях alpha и power, получим тот же самый размер выборки, что был ранее в качестве входного параметра:

Ранее было сказано, что эффекты ниже 0.125 будут обладать меньшей мощностью при 1004 наблюдениях. Следовательно, больше наблюдений → больше мощность для того же эффекта и выше. На Графике 2 видна зависимость MDE от размера выборки:

![](https://miro.medium.com/v2/resize:fit:700/1*AwHZT00xbRIhUa7ySyEG5Q.png)

График 2: по горизонтали — размер выборки на группу, по вертикали — MDE. Уровень значимости = 95%, мощность для 3 уровней. Кривая не совсем монотонно убывает, т.к. основана на живом тесте, где дисперсия не была константной.

Чтобы вам проще было понять физический смысл MDE, представьте что у вас есть микроскоп и линзы увеличивающие зум кратно (4х, 16х, 32х и т.п.). На 4х вы видите большие микроорганизмы, но этого зума все еще недостаточно, чтобы заметить более мелкую живность. Чем выше зум, тем пристальнее и детальнее представляется возможным их изучение. MDE — линза. Кратность зума здесь зависит от количества наблюдений.

В питоне есть пакет [statsmodels](https://www.statsmodels.org/dev/generated/statsmodels.stats.power.tt_ind_solve_power.html), который позволяет рассчитать MDE, выборку и остальные параметры. В R те же методы в пакете [pwr](https://cran.r-project.org/web/packages/pwr/pwr.pdf). Можно убедиться с помощью кода ниже, что наши расчеты верные:

В пакете доступны методы для популярных параметрических стат. критериев: корреляция Пирсона, t-тест, z-тест долей, anova и прочие другие. За нас уже все формулы притворили в код.

# Почему так важно смотреть на MDE и p-value, а не только p-value?

В каждом исследовании всегда есть вывод, о том что гипотеза отвергается или принимается. Сравнение P-value с альфой говорит лишь о наличии или отсутствии эффекта, но не говорит о его величине. Ориентируясь на Effect Size, мы также ориентируемся и на мощность. Всегда есть возможность оценить мощность эффекта по ходу или после завершения теста. Это развеет сомнения о его надежности и даст понимание, насколько сильно в эффект можно верить.

При репортинге также нельзя опускать такой важный элемент как доверительный интервал для эффекта. Эффект никогда не бывает константен и у него есть разброс. “Мы заработаем дополнительные 10 млн в год, благодаря новой фиче!”. Это не так. Благодаря фиче, дополнительная ценность будет находиться в пределах E[X]±SE рублей, в соответствии с выбранным пороговым значением уровня значимости. ДИ может перекрывать 0, что означает отсутствие значимого эффекта вовсе.

Многие также опускают еще один важный фактор — сезонность. Если ветка Б принесла дополнительные E[X]±SE рублей, допустим, осенью, то это не означает, что такой же размах экстраполируется на весь год.

Расчет Effect Size необходим и по той причине, что у бизнеса есть определенные временные рамки, чтобы выдерживать темп развития продукта. В пример сюда можно привести планирование на квартал, в рамках которого нужно разработать и протестировать N фичей.

Представьте, что вы запустили сразу несколько тестов, а p-value никак не пробьет альфу. Команда пребывает в непонятках что дальше делать. Зная Effect Size, появилась бы уверенность, что lift будет точно ниже достигнутой точки значения Effect Size. Это снизит уровень абстракции и даст возможность принять решение увереннее.

# Руководство и порядок действий

## Сколько времени ждать для достижения % изменения метрики?

**1.Определитесь с ключевой метрикой**, которую вы будете считать. Лучшая метрика — поюзерная. Почему?

- Эксперименты почти всегда направлены на восприятие пользователей
- В качестве ответа tt_ind_solve_power вы получаете количество наблюдений, необходимое для теста. Логично предположить, что если мы воздействуем на пользователей (а не на чеки или другие неодушевленные предметы), то наблюдениями правильно считать пользователей

**2. Посчитайте для этой метрики среднее и стандартное отклонение** по выборке за достаточно длительный период ( 2–5 недель). Это константы, которые будут использоваться далее в расчетах необходимой выборки на эксперимент. У кумулятивных метрик есть окно, которое нужно учитывать (про это ниже)

**3. Выберите lift от 0.01 до 1.00** (где 1.00–100% прироста у метрики). Стоит учитывать, что в зависимости от метрики у нас есть ограничения в наблюдениях и претендовать на маленькие эффекты не получится. Чтобы получить меньший эффект на тех же наблюдениях, необходимо повышать чувствительность метрики (см. [статью про CUPED](https://medium.com/statistics-experiments/cuped-%D0%B8%D0%BB%D0%B8-%D1%83%D0%B2%D0%B5%D0%BB%D0%B8%D1%87%D0%B5%D0%BD%D0%B8%D0%B5-%D1%87%D1%83%D0%B2%D1%81%D1%82%D0%B2%D0%B8%D1%82%D0%B5%D0%BB%D1%8C%D0%BD%D0%BE%D1%81%D1%82%D0%B8-%D0%BC%D0%B5%D1%82%D1%80%D0%B8%D0%BA%D0%B8-de7183fc964c))

**4. Рассчитайте Effect Size**: `effect_size = mean * lift / sd`

Как считать метрики со свойством накопления (например, arpu)

![](https://miro.medium.com/v2/resize:fit:700/1*LrOYeIZDmytdEYKDXvOb3g.png)

- ARPU можно отнести к User Average метрике, т.к. усредняется на пользователя. Она вполне подходит для прогноза необходимого количества наблюдений на эксперимент, но с нюансами.
- Метрика обладает свойством накопления и поэтому при расчете `effect_size` стоит иметь ввиду, что надо фиксировать время расчета. Пример накопления: arpu 1 week: mean = 10.1, sd = 2.1; arpu 2 week: mean = 19.3, sd = 2.3; arpu 3 week: mean = 25.1, sd = 2.6
- Если мы считаем arpu для 2 недель, то среднее фиксируем как 19.4, а ско как 2.3. Хотим увидеть лифт 1%. Тогда `effect_size = 19.4 * 0.01 / 2.3`
- При расчете effect_size учтите, что прогноз будет дан для изменения при ожидании 2 недель. Т.е. для 1 и 3 недель оценка по необходимому количеству наблюдений будет уже не актуальна (потому что значения средних и дисперсий другие). Решение: сделать несколько прогнозных отсечек и ориентироваться на соответствующее окно.

**Как считать** `sd`, `mean` **для ratio-метрики (например, AOV - средний чек)**

- Ratio-метрика — метрика отношения чего-то к чему-то. В числителе и знаменателе распределения случайных величин. В каждом из распределений своя дисперсия. Нужен метод, который позволит учесть обе дисперсии.
- Чтобы получить оценку sd для ratio-метрики, используйте дельта-метод. Про то как он работает, можно почитать в оригинальной [статье от Microsoft Exp-Platform](https://arxiv.org/pdf/1803.06336.pdf).
- Все также стоит иметь ввиду, что средние и дисперсию надо брать на временной горизонт. И также помнить про то, что метрики обладают эффектом накопления

**5. Выберите стат. параметры** уровня значимости α и уровня мощности 1-β

Рекомендации:

- Уровень мощности не менее 80%
- Уровень значимости не менее 95% (чем больше — тем лучше). Стоит учесть множественные поправки и FWER (family wise error rate). Используйте поправку Бонферонни: `alpha / m`, где `m` — количество рассматриваемых гипотез (парных сравнений). При таком отношении фиксируется изначальный уровень α

**6. Вставьте параметры в функцию для расчета выборки** из модуля `statmodels.stats`

**7. Рассчитайте время, исходя из траффика.** Возьмите полученное число и поделите на траффик за время t (где t — день/неделя/месяц)

- Тут стоит отметить, что брать DAU не совсем верно. Один и тот же пользователь может возвращаться на сайт/в приложение в течение недели/месяца. Время правильнее оценивать, принимая в расчет удержание пользователей
- Результатом будет необходимое количество наблюдений на одну выборку. Т.к. расчеты предполагают соотношение 50%/50% в группах А и Б, то полученное число нужно умножить на 2
- Если вы запускаете дисбалансированный тест не 50/50([см. статью про дисбаланс](https://medium.com/statistics-experiments/%D0%B4%D0%B8%D1%81%D0%B1%D0%B0%D0%BB%D0%B0%D0%BD%D1%81-%D0%B2-a-b-%D1%82%D0%B5%D1%81%D1%82%D0%B0%D1%85-%D0%B5%D1%81%D1%82%D1%8C-%D0%BB%D0%B8-%D1%80%D0%B0%D0%B7%D0%BD%D0%B8%D1%86%D0%B0-%D0%BC%D0%B5%D0%B6%D0%B4%D1%83-99-1-%D0%B8-50-50-%D0%B2-%D1%8D%D0%BA%D1%81%D0%BF%D0%B5%D1%80%D0%B8%D0%BC%D0%B5%D0%BD%D1%82%D0%B0%D1%85-11c8f4fe7eb4)), то регулируйте параметр ratio в соответствии с вашим балансом

## На какой эффект я могу рассчитывать, если у меня ограниченное количество наблюдений?

Такой кейс возможен, когда вы является подписочным сервисом и работаете с активацией после первой оплаты: ограниченная база и приток новых пользователей низкий. Другой пример: маленький б2б продукт, у которого немного клиентов и на этой базе не получится протестировать АБ с эффектами ниже какого-то значения.

У вас есть задача протестировать фичу Х. С фичей потенциально взаимодействует какое-то количество пользователей. Исходите из емкости подверженности (exposure) этой фичей (например, показ). Это и будет ориентиром.

1. **Определитесь с ключевой метрикой**, которую вы будете считать
2. **Посчитайте количество доступных наблюдений.** В случае с подпиской — сколько активных пользователей фичи Х?
3. **Посчитайте допустимый MDE для такого количества наблюдений.**

Функция `tt_ind_solve_power` возвращает значение для одного из 4 аргументов, который пропущен (None).

Поэтому, при известном размере выборки, уровне значимости и мощности, мы получаем в результате `effect_size`

# Завершение теста и подведение итогов

Настал момент подведения итогов или промежуточных итогов. На момент проверки все также необходимо ориентироваться на p-value. Далее MDE необходимо сравнить с фактическим Lift’ом и по следующим правилам огласить вердикт:

- **Не можем сказать, что метрика изменилась (MDE_EST < MDE_PRED).** По прошествии двух недель, MDE для ARPU дошел до уровня ~8%. Наблюдаемое изменение (lift) 3%. Мы можем посчитать на основе текущих данных фактический MDE_EST, и увидеть отличие от прогнозного. Это значит, что мы не можем подтвердить изменение lift
- **Успешное изменение метрики (MDE_EST ≥ MDE_PRED).** В первую неделю видим рост метрики ARPU до 15%, результаты стат. значимы. Наблюдаемый MDE на уровне 12%. Получается, что MDE_EST ≥ MDE_PRED и это значит, что мы видим реальный эффект. Стоит еще понимать, что необходимо брать модуль по лифту, т.е. MDE работает одинаково как в положительную, так и отрицательную стороны

Если на MDE смотреть не хочется, то можно ориентироваться на уровень мощности. С точки зрения интерпретации ничего не меняется: если мощность ниже установленного порога (например, 80%), то необходимо больше данных для наблюдаемого lift’а

# Заключение

Выше было рассмотрено, как стоит подходить к планированию времени на проведение эксперимента. Помимо этого, порядок анализа эксперимента на пост-периоде также меняется, если раньше вам приходилось смотреть только на p-value.

Приведенные в этом руководстве инструменты работают с параметрическим подходом, когда нам известно с каким параметром распределения предстоит столкнуться (например, дисперсия и среднее). В следующей части руководства будут рассмотрены альтернативные инструменты, которые помогут определить время эксперимента для специфичной метрики продукта, не поддающаяся описанию известными функциями распределений.