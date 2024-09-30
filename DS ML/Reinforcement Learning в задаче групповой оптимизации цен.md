---
tags:
  - data
author: X5
link: https://habr.com/ru/companies/X5Tech/articles/826400/
source: habr
data_type:
  - DS
company: X5
---
### Введение

Привет, Хабр! На связи аналитики больших данных Х5 Tech — [Антон Денисов](https://www.linkedin.com/in/anton-denisov-71b98715a/) и [Михаил Будылин](https://ru.linkedin.com/in/mikhail-budylin-15b25220a). В предыдущей [статье про многоруких бандитов](https://habr.com/ru/companies/X5Tech/articles/783390/) мы показали, как можно искать оптимальную цену в разрезе одного товара. На практике можно встретить ограничение на группу товаров, например, товары категории не должны быть в среднем дороже, чем товары конкурентов. Такое ограничение иногда называют ценовым индексом.

Методы решения задач с общими ограничениями мы разбирали в [статье про оптимизационные задачи](https://habr.com/ru/companies/X5Tech/articles/685590/), но в тех подходах сложность заключается в том, что зависимость спроса от цены задаётся моделью эластичности, которую ещё надо уметь хорошо аппроксимировать на исторических данных.

Цель данной статьи – исследовать метод, комбинирующий подход обучения с подкреплением и оптимизатором, учитывающем общее ограничение на цены.

### Оптимизационная задача

#### Обозначения

Введём следующие обозначения:

![n](https://habrastorage.org/getpro/habr/formulas/7/7b/7b8/7b8b965ad4bca0e41ab51de7b31363a1.svg) - количество товаров,

![m](https://habrastorage.org/getpro/habr/formulas/6/6f/6f8/6f8f57715090da2632453988d9a1501b.svg) - количество перебираемых цен для каждого товара ![i=\overline{1,n}](https://habrastorage.org/getpro/habr/formulas/c/cf/cf5/cf5f0c82ebe1b30c7a7aa9b16a08fb90.svg),

![p \in \mathbb{R}^{n \times m}](https://habrastorage.org/getpro/habr/upload_files/29b/3b6/474/29b3b647429c489dd0c768cf635912ce.svg), ![p_{i, j}](https://habrastorage.org/getpro/habr/formulas/3/31/315/3151736d0b6c6420a8cf006146d2f9db.svg) - ![j](https://habrastorage.org/getpro/habr/formulas/3/36/363/363b122c528f54df4a0446b6bab05515.svg) - ая цена товара ![i](https://habrastorage.org/getpro/habr/formulas/8/86/865/865c0c0b4ab0e063e5caa3387c1a8741.svg),

![p^{(c)} \in \mathbb{R}^{n}](https://habrastorage.org/getpro/habr/formulas/1/1d/1d2/1d2264fe73f25a8e01aa417c0af3b0d9.svg), ![p^{(c)}_{i}](https://habrastorage.org/getpro/habr/formulas/1/1a/1a6/1a629dbe30781abf0c7fc10dbc41c8c5.svg) - средне-рыночная цена товара ![i](https://habrastorage.org/getpro/habr/formulas/8/86/865/865c0c0b4ab0e063e5caa3387c1a8741.svg),

![c_{i}](https://habrastorage.org/getpro/habr/formulas/d/d9/d98/d9899588b2b28a768a63ade0f3523596.svg) - себестоимость товара ![i](https://habrastorage.org/getpro/habr/formulas/8/86/865/865c0c0b4ab0e063e5caa3387c1a8741.svg),

![I = \frac{1}{n} \sum_{i=1}^{n} \frac{p_i}{p^{(c)}_{i}}](https://habrastorage.org/getpro/habr/formulas/6/64/643/6430a5f30d6827682a4f790807899e40.svg) - рыночный индекс с ценами сети ![p_{i}](https://habrastorage.org/getpro/habr/formulas/8/8a/8a4/8a4bbd153c74655abb7ca04c0fa901d8.svg) и ценами рынка ![p^{(c)}_{i}](https://habrastorage.org/getpro/habr/formulas/1/1a/1a6/1a629dbe30781abf0c7fc10dbc41c8c5.svg) для товаров  
![i=\overline{1, n}](https://habrastorage.org/getpro/habr/formulas/b/b5/b5f/b5f1b1713b6fe7823933a5518739d159.svg)

![I_{l}, I_{u}](https://habrastorage.org/getpro/habr/formulas/3/35/358/358f09c167fe48c7583e1fc201078d61.svg) - нижняя и верхняя границы индекса по рынку, устанавливаемые экспертно или из исторических данных.

![q \in \mathbb{R}^{n \times m}](https://habrastorage.org/getpro/habr/formulas/2/27/279/279d56a25576f319a9ba5cb530cc51ae.svg), ![q_{i, j}](https://habrastorage.org/getpro/habr/formulas/6/65/65c/65c5c81df2176d5230b4a6c288f63f2d.svg) - с.в. продаж товара ![i](https://habrastorage.org/getpro/habr/formulas/8/86/865/865c0c0b4ab0e063e5caa3387c1a8741.svg) по цене ![j](https://habrastorage.org/getpro/habr/formulas/3/36/363/363b122c528f54df4a0446b6bab05515.svg) с распределением ![Poisson(\lambda_{i, j})](https://habrastorage.org/getpro/habr/formulas/7/70/70b/70b2bb758c44c00950cdb3a0c02d4b72.svg),

![\lambda \in \mathbb{R}^{n \times m}](https://habrastorage.org/getpro/habr/formulas/4/41/41a/41a9775958ff19dee203b04977afb900.svg) , ![\lambda_{i, j}](https://habrastorage.org/getpro/habr/formulas/d/de/ded/ded5224b6a92b649a7fc128e2a0461fc.svg) - мат. ожидание продаж товара ![i](https://habrastorage.org/getpro/habr/formulas/8/86/865/865c0c0b4ab0e063e5caa3387c1a8741.svg), по цене ![j](https://habrastorage.org/getpro/habr/formulas/3/36/363/363b122c528f54df4a0446b6bab05515.svg).

#### Постановка задачи

Будем оптимизировать ожидаемую валовую прибыль, целевая функция которой записывается следующим образом:

![M = \mathbb{E} \sum_{i=1}^{n}\sum_{j=1}^{m} (p_{i,j}- с_{i}) \cdot q_{i,j} \cdot x_{i,j} \to \max_{x} \qquad \qquad (1)](https://habrastorage.org/getpro/habr/upload_files/571/9e9/1c8/5719e91c8c63fbb781156c412d2aa082.svg)

Для каждого товара выбирается ровно одна цена из набора(![x_{i, j} \in \{0, 1\}](https://habrastorage.org/getpro/habr/upload_files/33f/146/144/33f14614418a841b20f9bcf80411c19a.svg)):

![\sum_{j=1}^{m} x_{i, j} = 1,   \ i=\overline{1, n} \qquad \qquad \qquad (2)](https://habrastorage.org/getpro/habr/upload_files/2bc/b23/acc/2bcb23acc605d8af81688dca6de688e3.svg)

Рыночное позиционирование (иногда называемое ценовым индексом) ограничено сверху и снизу, оно описывается следующим неравенством:

![I_{l} \leqslant \sum_{i=1}^{n} \sum_{j=1}^{m} \frac{p_{i, j}}{p_{i}^{(c)}} x_{i, j} \leqslant I_{u} \qquad \qquad (3)](https://habrastorage.org/getpro/habr/upload_files/3d5/b98/271/3d5b982711baacfd3dd427fe8fd6cb15.svg)

В данной постановке предполагается, что для каждого товара задан допустимый набор цен для перебора. Этот набор может формироваться из исторических данных, плановых показателей, рыночных условий или экспертно.

### Примеры используемых модельных данных

#### Таблица 1. Набор цен и себестоимостей для поиска цен группы товаров

Чтобы максимально упростить расчёты, будем использовать небольшой пример с пятью товарами и сеткой из 4-х цен, а также среднее значение цены конкурента и себестоимости:

|   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|
||![p_1](https://habrastorage.org/getpro/habr/upload_files/e27/b62/e82/e27b62e825ba230e29a8233417b64bb2.svg)|![p_2](https://habrastorage.org/getpro/habr/formulas/6/6f/6fe/6fe97b358b528edc477ba63d50b652af.svg)|![p_3](https://habrastorage.org/getpro/habr/upload_files/ff6/23b/155/ff623b1554b2b17ee58d860eaf1b58c7.svg)|![p_4](https://habrastorage.org/getpro/habr/formulas/0/0c/0c9/0c96eb903a72377d321d97c8e07f9ae6.svg)|![p^{(c)}](https://habrastorage.org/getpro/habr/upload_files/5d5/a14/eb6/5d5a14eb623ce1c76609533f6ba15553.svg)|![c](https://habrastorage.org/getpro/habr/formulas/4/4a/4a8/4a8a08f09d37b73795649038408b5f33.svg)|
|товар 1|100|110|120|130|105|80|
|товар 2|50|55|60|65|60|45|
|товар 3|10|11|12|13|11|9|
|товар 4|40|45|50|55|40|39|
|товар 5|70|75|80|85|80|65|

#### Таблица 2. Набор значений спроса для заданной сетки цен

Средние значения спроса для каждой цены на каком-то историческом промежутке зададим в следующем виде:

|   |   |   |   |   |
|---|---|---|---|---|
||![\lambda(p_1)](https://habrastorage.org/getpro/habr/upload_files/e29/115/c68/e29115c6829a072857c09cf2ed05fcb3.svg)|![\lambda(p_2)](https://habrastorage.org/getpro/habr/formulas/e/e4/e4a/e4af3069773f4e5671490624fbc090ef.svg)|![\lambda(p_3)](https://habrastorage.org/getpro/habr/formulas/f/fd/fd9/fd93c3aa6001122bf21ef251ea0f4745.svg)|![\lambda(p_4)](https://habrastorage.org/getpro/habr/formulas/6/69/698/6983d36a5da3c116a214a593b4da0717.svg)|
|товар 1|4.0|3.5|3.0|2.0|
|товар 2|3.0|2.0|2.0|1.0|
|товар 3|7.0|5.0|4.0|2.8|
|товар 4|9.0|8.6|8.3|8.0|
|товар 5|3.0|2.6|2.0|1.3|

#### Решение оптимизационной задачи (1-3) для данных таблиц (1-2)

Задача (1-3) является задачей целочисленного линейного программирования, которая будет решаться с помощью библиотеки [Pyomo](https://habr.com/ru/companies/X5Tech/articles/708294/) в Python и солвера [Highs](https://highs.dev/).

Далее будем сравнивать задачу без ограничения (1-2) с задачей с ограничением (1-3).

Оптимальное решение задач для приведённых в Таблицах (1-2) значений **с точки зрения целевой функции и индекса**:

задача (1-2): ![p_{opt} = [120,\ 60,\ 12,\ 55,\ 80]](https://habrastorage.org/getpro/habr/upload_files/106/339/529/1063395290b5ee72313d6cc65d7dfc8f.svg); ![M_{opt} = 320.0](https://habrastorage.org/getpro/habr/upload_files/3b3/982/81a/3b398281a1afbe337693e43d959dc129.svg); ![I_{opt} = 1.122](https://habrastorage.org/getpro/habr/upload_files/54f/71d/283/54f71d283951990ba09a91f7ff4ace9a.svg);

задача (1-3): ![p_{opt} = [110,\ 50,\ 10,\ 55,\ 70]](https://habrastorage.org/getpro/habr/upload_files/268/afe/858/268afe858bacd88f43fcbb48b9ddd51b.svg); ![M_{opt} = 270.0](https://habrastorage.org/getpro/habr/formulas/5/51/51c/51c8822128dcc20931104086d5a5384f.svg); ![I_{opt} = 1.008](https://habrastorage.org/getpro/habr/formulas/b/b3/b31/b31c49ca3f59c1b3d55dd2b549f73038.svg).

Полученные значения мы будем использовать для сравнения с результатами моделирования.

## Схема экспериментов с RL

Стратегией поиска решения будет алгоритм сэмплирования [Томпсона](https://en.wikipedia.org/wiki/Thompson_sampling).

Процесс обновления параметров похож на описанный [ранее](https://habr.com/ru/companies/X5Tech/articles/783390/#:~:text=%D0%92%20%D1%8D%D0%BA%D1%81%D0%BF%D0%B5%D1%80%D0%B8%D0%BC%D0%B5%D0%BD%D1%82%D0%B0%D1%85%20%D0%B4%D0%BB%D1%8F,%D0%BE%D0%B7%D0%BD%D0%B0%D0%BA%D0%BE%D0%BC%D0%B8%D1%82%D1%8C%D1%81%D1%8F%20%D1%82%D1%83%D1%82) в случае поиска лучшей цены для одного товара. Отличие в том, что на каждой итерации после шага сэмплирования решается описанная выше оптимизационная задача (1-2) или (1-3).

Схема алгоритма:

![Рис. 1. Схема алгоритма сэмплирования с оптимизацией](https://habrastorage.org/r/w1560/getpro/habr/upload_files/b0c/8a8/8c0/b0c8a88c078305a93871d9a4edb75daf.png "Рис. 1. Схема алгоритма сэмплирования с оптимизацией")

Рис. 1. Схема алгоритма сэмплирования с оптимизацией

## Результаты моделирования

### Параметры экспериментов

Для запуска моделей будем использовать:

- сетку перебираемых цен ![p](https://habrastorage.org/getpro/habr/formulas/8/83/838/83878c91171338902e0fe0fb97a8c47a.svg) (таблица 1);
    
- таблицу значений спроса ![\lambda](https://habrastorage.org/getpro/habr/formulas/c/c6/c6a/c6a6eb61fd9c6c913da73b3642ca147d.svg) (таблица 2);
    
- значения для границ рыночного индекса ![I_{l}](https://habrastorage.org/getpro/habr/formulas/f/fd/fd7/fd7843d4d9aaef5ea9e10f85107d2e20.svg) и ![I_{u}](https://habrastorage.org/getpro/habr/formulas/a/a2/a2b/a2b0882fadce4c20d84e4665f61ecc17.svg)равны, соответственно, 0.98 и 1.02.
    

Параметры априорного распределения спроса ![\lambda_{i,j}](https://habrastorage.org/getpro/habr/formulas/3/32/32f/32f653868da87655fdf0dd43d1f6a4c4.svg) ~ ![Gamma(\alpha_{i, j}, \beta_{i, j})](https://habrastorage.org/getpro/habr/formulas/0/08/082/082ba0a74d6dbed99706b2f941df7c4d.svg), которое является сопряжённым к распределению Пуассона, задаются следующими способами:

- Задаём одинаковое значение ![\alpha](https://habrastorage.org/getpro/habr/formulas/7/7b/7b7/7b7f9dbfea05c83784f8b85149852f08.svg) из набора 2, 4, 12, а параметр ![\beta = 1](https://habrastorage.org/getpro/habr/formulas/7/75/75b/75bb47bb9659df45d3f1b43362638d7c.svg);
    
- Для каждого товара оцениваем ![\alpha_{i}](https://habrastorage.org/getpro/habr/formulas/0/07/07c/07c9f0f65f673a264a5101683e774507.svg) на предыстории продаж в 30 шагов, считая, что для каждого товара установлена цена ![p_{i, 2}](https://habrastorage.org/getpro/habr/formulas/f/fc/fc5/fc5a063ef8fe5a77f65510bb666dce28.svg), а ![\beta_{i, j} = 1.](https://habrastorage.org/getpro/habr/formulas/4/4e/4e5/4e598edcfe031116ddfa9a3e9f97d0dd.svg)
    

### Метрики для анализа

Поведение многоруких бандитов для предложенной схемы решения задачи будем исследовать с помощью следующих показателей:

- ![M_{k} = \mathbb{E} M_{opt, k}](https://habrastorage.org/getpro/habr/formulas/8/8f/8f7/8f7a8d42cf917090203b926f197d978d.svg) - мат.ожидание оптимального решения с помощью многорукого бандита на шаге ![k](https://habrastorage.org/getpro/habr/formulas/8/8c/8ce/8ce4b16b22b58894aa86c421e8759df3.svg).
    
- ![Regret_{k} = \mathbb{E} [M_{opt} - M_{k}]](https://habrastorage.org/getpro/habr/formulas/6/61/615/61580c5f96e0dd2726b446abeda2106d.svg) - мат.ожидание отклонения решения, полученного с помощью алгоритма от теоретического оптимального значения целевой функции на шаге ![k](https://habrastorage.org/getpro/habr/formulas/8/8c/8ce/8ce4b16b22b58894aa86c421e8759df3.svg).
    
- ![CumulativeRegret_{k} = \sum_{k'=1}^{k} Regret_{k'}](https://habrastorage.org/getpro/habr/formulas/e/e3/e3b/e3bc91943f4f9f06c78df1384a7aaa18.svg) - накопленный к шагу ![k](https://habrastorage.org/getpro/habr/formulas/8/8c/8ce/8ce4b16b22b58894aa86c421e8759df3.svg) ![Regret](https://habrastorage.org/getpro/habr/formulas/3/31/314/3141af4744f17a254e58c913ae90281e.svg).
    
- ![I_{k} = \mathbb{E}\frac{1}{n} \sum_{i=1}^{n} \frac{p_{i}^{(opt, k)}}{p_{i}^{(c)}}](https://habrastorage.org/getpro/habr/upload_files/5a7/f60/83b/5a7f6083bd8ef107f4ef50034a564d34.svg) - мат.ожидание ценового индекса оптимального решения ![p^{(opt, k)}](https://habrastorage.org/getpro/habr/formulas/f/f3/f34/f347e4750d12b85a6c64566fa5e84778.svg) на шаге ![k](https://habrastorage.org/getpro/habr/formulas/8/8c/8ce/8ce4b16b22b58894aa86c421e8759df3.svg).
    

### Динамика регрета при различных параметрах инициализации

Сравним поведение ![Regret](https://habrastorage.org/getpro/habr/formulas/3/31/314/3141af4744f17a254e58c913ae90281e.svg) для каждой из задач (1-2) (без ограничения на индекс) и (1-3) (с ограничением на индекс) в зависимости от начальной инициализации априорного распределения оценки спроса ![\lambda.](https://habrastorage.org/getpro/habr/formulas/5/52/521/5217a9eb510dcda6ac122fe1bfbc3b8e.svg)  
Значение pre_30 соответствует среднему значению на истории, взятому за 30 шагов до начала запуска алгоритма при цене ![p_{2}](https://habrastorage.org/getpro/habr/formulas/b/bb/bbd/bbdc52880341d0aeb3b3006af0a6583a.svg).

![Рис. 2. Динамика Regret для задачи без ограничения на индекс](https://habrastorage.org/r/w1560/getpro/habr/upload_files/3bd/27c/dab/3bd27cdab6ca603f4ec015129bd0ee9f.png "Рис. 2. Динамика Regret для задачи без ограничения на индекс")

Рис. 2. Динамика Regret для задачи без ограничения на индекс

![Рис. 3. Динамика Regret для задачи с ограничением на индекс](https://habrastorage.org/r/w1560/getpro/habr/upload_files/747/40f/9ee/74740f9ee9cd4012cc4e608bfa12fa5a.png "Рис. 3. Динамика Regret для задачи с ограничением на индекс")

Рис. 3. Динамика Regret для задачи с ограничением на индекс

![Рис. 4. Динамика накопленного Regret для задачи без ограничения на индекс](https://habrastorage.org/r/w1560/getpro/habr/upload_files/e6e/df0/ba9/e6edf0ba919cade5b7419326c5af2295.png "Рис. 4. Динамика накопленного Regret для задачи без ограничения на индекс")

Рис. 4. Динамика накопленного Regret для задачи без ограничения на индекс

![Рис. 5. Динамика накопленного Regret для задачи с ограничением на индекс](https://habrastorage.org/r/w1560/getpro/habr/upload_files/2b4/0a1/509/2b40a1509cf834357de61b060a104c02.png "Рис. 5. Динамика накопленного Regret для задачи с ограничением на индекс")

Рис. 5. Динамика накопленного Regret для задачи с ограничением на индекс

Можно наблюдать, что наилучшие показатели в задаче с ограничениями мы получаем при инициализации _pre_30_, что можно объяснить тем, что в начале мы имеем априорные распределения, близкие к истинным, поэтому алгоритм тратит меньше шагов на исследование и уточнение значений спроса.

Для задачи без ограничения влияние начальной аппроксимации 2, 4, _pre_30_ незначительно, при 12 - можно наблюдать, что результаты значительно уступают, это связано в основном с тем, что 12 - это значение, которое сильно отличается от истинных для большинства товаров, что приводит к более длительному уточнению истинных значений спроса.

Далее будем исследовать динамику при инициализации pre_30, это можно обосновать тем, что в реальности мы так или иначе для большинства товаров имеем продажи, которые можно использовать для начальной аппроксимации спроса.

![Рис. 6. Сравнение динамики Regret для двух типов задач](https://habrastorage.org/r/w1560/getpro/habr/upload_files/fa9/3e0/735/fa93e07350fe2e04a7838315675fb264.png "Рис. 6. Сравнение динамики Regret для двух типов задач")

Рис. 6. Сравнение динамики Regret для двух типов задач

Regret вначале чуть ниже для задачи с ограничениями, что связано с более узким множеством вариантов перебора.

### Динамика целевого показателя

![Рис. 7. Динамика валовой прибыли](https://habrastorage.org/r/w1560/getpro/habr/upload_files/bf0/3ff/cdc/bf03ffcdc4ee1ddc4e33e988b3d58ebc.png "Рис. 7. Динамика валовой прибыли")

Рис. 7. Динамика валовой прибыли

Скорости выхода на оптимум в обеих задачах похожи, в начале работы алгоритма ограничения сужают область перебора, что повышает скорость сходимости до первой сотни итераций.

### Динамика ценового индекса

![Рис. 8. Динамика среднего ценового индекса](https://habrastorage.org/r/w1560/getpro/habr/upload_files/435/98f/0d2/43598f0d2adc920a42c7efd7a0556658.png "Рис. 8. Динамика среднего ценового индекса")

Рис. 8. Динамика среднего ценового индекса

Как мы видим по динамике индекса, на начальных шагах он поднимается близко к верхней границе. Связано это с тем, что априорные оценки продаж по каждому товару задаются одинаково для всех цен одного товара, то есть заведомо на старте алгоритм приоритетно выбирает наибольшие цены, (![(p-c) \cdot q](https://habrastorage.org/getpro/habr/formulas/4/48/486/486366e3d3316699c365452b0dd6e72f.svg) (![q = const](https://habrastorage.org/getpro/habr/formulas/5/52/525/5253b3abb51662d41d00d11d9f7df852.svg)) – очевидно, что максимум достигается при наибольшей допустимой цене с учётом ограничений). После исследования области с высокой ценой алгоритм переходит на поиск в области более низких цен, и постепенно это приводит к понижению индекса с выходом на теоретическое значение.

### Выбор оптимального решения

Последний вопрос, который мы разберём в этой статье – как выбрать окончательное оптимальное решение? Опишем два подхода выбора финального решения.

1. Останавливаем алгоритм поиска цен, когда доля определённого решения (ожидаемо самого оптимального) на последних шагах не ниже заданного уровня.
    
2. Выбираем решение, которое используется алгоритмом чаще всего на последних шагах.
    

Оба подхода обоснованы тем, что в ходе работы алгоритма всё чаще и чаще выбирается наилучшее решение, это мы можем наблюдать на графиках Regret'а на рис. (2-3).  
Будем использовать второй подход – выбор наиболее частого решения алгоритма за последние ![k](https://habrastorage.org/getpro/habr/formulas/8/8c/8ce/8ce4b16b22b58894aa86c421e8759df3.svg) шагов.

Сравним статистику отклонения выбранного решения от истинного оптимального (4-5) для каждой из задач с инициализацией pre_30 по значению валовой прибыли. Общее количество шагов для выбора лучшего решения выберем 1000, а период последних шагов – 100.

![Рис. 9. Распределение дельты валовой прибыли для найденного решения с  оптимальным для задачи без ограничения](https://habrastorage.org/r/w1560/getpro/habr/upload_files/e31/fb6/d7b/e31fb6d7b978dfbf05404b6195bd2e74.png "Рис. 9. Распределение дельты валовой прибыли для найденного решения с  оптимальным для задачи без ограничения")

Рис. 9. Распределение дельты валовой прибыли для найденного решения с оптимальным для задачи без ограничения

![Рис. 10. Распределение дельты валовой прибыли для найденного решения с  оптимальным для задачи с ограничением](https://habrastorage.org/r/w1560/getpro/habr/upload_files/592/3fa/da1/5923fada15d8bd45717dca21ea4f95b6.png "Рис. 10. Распределение дельты валовой прибыли для найденного решения с  оптимальным для задачи с ограничением")

Рис. 10. Распределение дельты валовой прибыли для найденного решения с оптимальным для задачи с ограничением

Как мы можем наблюдать на рис. (9-10), критерий выбора решения может выбрать субоптимальное решение с ненулевой вероятностью.

В моделируемых экспериментах доля выбора оптимального значения (которую можно считать оценкой вероятности выбора оптимума) 0.931 для задачи (1-2) и 0.929 для задачи (1-3).

## Выводы

В данной статье мы:

- показали, как можно скомбинировать оптимизатор с обучением с подкреплением для поиска решения с общим ограничением на переменные и без;
    
- провели моделирование на примере небольшой задачи, изучили поведение специфических для таких задач метрик;
    
- описали способ определения итогового оптимального решения;
    
- установили, что решение задач сходится к оптимуму, а наличие истории оказывает положительное влияние на скорость сходимости.