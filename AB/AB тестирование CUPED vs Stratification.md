---
tags:
  - data
link: https://habr.com/ru/companies/X5Tech/articles/826488/
source: habr
author: Nazarov
data_type:
  - AB tests
company: X5
---
Хабр, привет! Сегодня рассмотрим кейс, в котором классические статистические критерии не работают, и разберёмся, почему так происходит. Научимся строить свои собственные критерии по историческим данным. Обсудим плюсы и минусы такого подхода.

Меня зовут [Коля](http://www.linkedin.com/in/nazarovn), я работаю аналитиком данных в X5 Tech. Мы с [Сашей](http://www.linkedin.com/in/amsakhnov) продолжаем писать серию статей по А/Б тестированию. Это наша седьмая статья, предыдущие статьи можно найти в описании [профиля](https://habr.com/ru/users/nnazarov/).

## Маленькие выборки из скошенных распределений

В этой статье будем рассматривать эксперименты на маленьких выборках из скошенных распределений. Скошенное распределение — это асимметричное распределение, у которого один из хвостов более тяжёлый. Пример скошенного распределения — логнормальное распределение. Формула и графики его функции плотности распределения представлены ниже.

![f(x) = \dfrac{1}{x \sigma \sqrt{2 \pi}} e^{-\frac{(\ln x - \mu)^2}{2 \sigma^2}} \qquad \text{, где }x>0,\ \sigma > 0,\ \mu\in\mathbb{R}](https://habrastorage.org/getpro/habr/upload_files/97c/e29/6da/97ce296daf99e367f98bfa1f938b67e0.svg)

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/1c0/7b1/c42/1c07b1c4232b7515559bc94cadee96ac.png)

Маленькая выборка — понятие относительное. Дадим неформальное определение, которое будем использовать в рамках этой статьи. Будем называть выборку маленькой, если распределение её среднего также имеет скошенное распределение.

Распределение среднего зависит от размера выборки. Если сгенерировать выборку размера 1 из скошенного распределения и посчитать среднее, то это среднее будет иметь такое же скошенное распределение. Если сгенерировать выборку огромного размера из скошенного распределения, то по ЦПТ распределение среднего будем стремиться к нормальному распределению. Продемонстрируем это на выборках размеров 1, 100 и 100 000. Данные будем генерировать из логнормального распределения с параметрами ![\mu=0, \sigma=2](https://habrastorage.org/getpro/habr/upload_files/174/963/d13/174963d13058a835df60acb1a21e12f9.svg). На графиках ниже изображены оценки функций плотностей распределений среднего для разных размеров выборок.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/5a4/dc8/6a1/5a4dc86a158d6d3a57263caad9e325c2.png)

Почему мы решили рассмотреть эксперименты на маленьких скошенных выборках? Оказывается, классические критерии, которые используются для проверки гипотезы о равенстве средних, работают некорректно на таких данных (про корректность А/Б тестов можно почитать в этой [статье](https://habr.com/ru/company/X5Tech/blog/706388/)). Продемонстрируем это с помощью синтетических А/А тестов. Будем генерировать выборки размера 10 из логнормального распределения и проверять гипотезу о равенстве средних тестом Стьюдента. Повторим эту процедуру 10 000 раз и построим распределение p-value.

Код

```

```

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/a5a/32c/0d2/a5a32c0d2da64f9688156a5c2265bc75.png)

Распределение p-value оказалось неравномерным, критерий работает некорректно. При выводе теста Стьюдента используется предположение о нормальности распределения средних. Выше мы показали, что средние распределены не нормально. Вероятно, из-за этого критерий работает некорректно. Если увеличить размер выборок до 1 миллиона, то распределение средних будет близко к равномерному, и тест Стьюдента станет работать корректно.

Если в примере выше заменить критерий Стьюдента на критерий Манна-Уитни, то распределение p-value на синтетических А/А тестах станет равномерным. Однако, критерий Манна-Уитни в общем случае не проверяет гипотезу о равенстве средних. Это может приводить к противоречивым результатам. Например, для двух выборок с одинаковым средним критерий Манна-Уитни может отклонять нулевую гипотезу. Увеличение размера выборок не помогает решить эту проблему.

```
a = np.hstack([np.linspace(0, 1, 30), np.linspace(8, 9, 10)])b = np.linspace(2, 3, 40)pvalue = stats.mannwhitneyu(a, b).pvalueprint(f'разница средних = {a.mean() - b.mean()}')print(f'pvalue = {pvalue:0.5f}')
```

```
разница средних = 0.0pvalue = 0.00012
```

Бутстреп для маленьких выборок из скошенных распределений тоже работает некорректно. Бутстреп строит оценку на основе данных эксперимента, которых не хватает для достаточно точного описания распределения статистики, имеющей особенности. Подробнее про бутстреп можно прочитать в этой [статье](https://habr.com/ru/companies/X5Tech/articles/679842/).

## Как устроены статистические критерии

Мы предлагаем построить свой критерий для оценки подобных экспериментов. Сначала разберёмся, как вообще устроены статистические критерии. Вспомним ряд определений из математической статистики.

**Выборка** — набор случайных величин. В теории вероятностей и статистике отдельно выделяют случай независимых одинаково распределённых случайных величин (н.о.р.с.в.). Каждая из них распределена так же, как и остальные, и все они независимы в совокупности. Выборка из распределения F обозначается так: ![X_1, \ldots, X_n \sim F](https://habrastorage.org/getpro/habr/upload_files/88d/acf/249/88dacf249c7f8cd538da1b59b3e30209.svg) или ![X^n \sim F](https://habrastorage.org/getpro/habr/upload_files/391/fa9/47c/391fa947c14a0c992dbd85ca8f8ff8b0.svg).

**Статистика** — любая измеримая функция от выборки. Примеры статистик: минимум, среднее, квантиль. Статистику от выборки часто обозначают как ![T(X_1, \dots, X_n)](https://habrastorage.org/getpro/habr/upload_files/cba/f81/db3/cbaf81db3153d171a93ca9292e94ee05.svg).  

**Статистическая гипотеза** — любое предположение о распределении и свойствах случайной величины. Примеры статистических гипотез: выборка из нормального распределения, средние выборок равны. 

**Уровень значимости** — вероятность отклонить нулевую гипотезу при условии её истинности. Обозначается как ![\alpha](https://habrastorage.org/getpro/habr/upload_files/2c8/830/8e6/2c88308e61239388c678d0c2f2fd605c.svg).

**Статистическая мощность** — вероятность отклонения нулевой гипотезы в случае, когда альтернативная гипотеза верна.

**Статистический критерий** — математическое правило, позволяющее по реализациям выборок отвергнуть или не отвергнуть нулевую гипотезу с заданным уровнем значимости.

**Критическая область** — множество значений статистики критерия, при которых нулевая гипотеза отвергается.

Как строят статистические критерии? Обычно, для проверки нулевой гипотезы ![H_0](https://habrastorage.org/getpro/habr/upload_files/b8c/153/5c1/b8c1535c16074f4a5577e6e14c69d557.svg) против альтернативной гипотезы ![H_1](https://habrastorage.org/getpro/habr/upload_files/fda/e7f/ab7/fdae7fab7adf5780ceb7a372c086e5a6.svg) выбирается некоторая статистика ![T=T(X^n)](https://habrastorage.org/getpro/habr/upload_files/dc2/c65/bcc/dc2c65bcc430a2e5a61bf7f679d634a9.svg), обладающая двумя свойствами:

- Если ![H_0](https://habrastorage.org/getpro/habr/upload_files/992/3a5/0fb/9923a50fb121b3d7dd9bccd32ef63677.svg) верна, то статистика ![T](https://habrastorage.org/getpro/habr/upload_files/5b7/8e3/645/5b78e3645c8897de391984ff7eab3aef.svg) имеет некоторое непрерывное распределение ![G](https://habrastorage.org/getpro/habr/upload_files/ccf/2f1/521/ccf2f152147343a083b3f9f3965bf3ea.svg), в пределе не зависящее от ![n](https://habrastorage.org/getpro/habr/upload_files/bde/b68/5ee/bdeb685ee27c10524f9e50b6e2b05896.svg);
    
- Если ![H_0](https://habrastorage.org/getpro/habr/upload_files/750/eee/056/750eee05670d862841b5ec3ec06f03f0.svg) неверна, то ![|T (X^n)| \overset{P}{\to} \infty](https://habrastorage.org/getpro/habr/upload_files/d81/a12/0e9/d81a120e90c001f68b3342e779e5416c.svg) при ![n \to \infty](https://habrastorage.org/getpro/habr/upload_files/fae/a92/4de/faea924de33a1124e9923890d3471154.svg).
    

Зная распределение статистики при верности нулевой гипотезы ![G](https://habrastorage.org/getpro/habr/upload_files/855/909/5e7/8559095e7389c48544c030a8849d0d33.svg), можно определить область ![W](https://habrastorage.org/getpro/habr/upload_files/2b0/26b/68b/2b026b68b271285a29509d310c834fcb.svg), в которую статистика ![T](https://habrastorage.org/getpro/habr/upload_files/314/70d/f54/31470df545e598f601c3704f304b428b.svg) попадает с вероятность ![1-\alpha](https://habrastorage.org/getpro/habr/upload_files/4e4/841/8d5/4e48418d5f93e28e780cf8b45a877c57.svg). Конкретные границы области ![W](https://habrastorage.org/getpro/habr/upload_files/fe7/71b/00d/fe771b00dd6e4e0379beea9e7fba1ea8.svg) выбирают так, чтобы мощность критерия была максимальной. Если значение статистики попадает в область W, то нулевая гипотеза не отклоняется, иначе значение статистики попадает в критическую область, и нулевая гипотеза отклоняется.

Ниже на картинке приведён пример распределения G и области W для теста Стьюдента с ![\alpha=0.05](https://habrastorage.org/getpro/habr/upload_files/e92/0ac/6de/e920ac6de7e63679b67833d2a59b99ec.svg).

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/e96/220/b6b/e96220b6b55f19d064eb7772de0bb4d9.png)

Чтобы применять критерий, нужно знать распределение ![G](https://habrastorage.org/getpro/habr/upload_files/95f/1ca/30a/95f1ca30a6765c701353646b93358ede.svg). Для этого математики доказывают теоремы, что для определённой гипотезы при некоторых предположениях распределение ![G](https://habrastorage.org/getpro/habr/upload_files/12d/eb0/b21/12deb0b21de4b78d3381ef778445fe9c.svg) имеет такую-то функцию распределения. Если условия теоремы не выполняются, то критерий может работать некорректно.

## Строим свой критерий

Чтобы построить критерий для проверки гипотезы о равенстве средних, нужно определить статистику и узнать её распределение при верности нулевой гипотезы. В качестве статистики логично использовать разность средних. Оценить распределение статистики можно по историческим данным с помощью синтетических А/А тестов. Для каждого синтетического А/А теста вычислим и запомним значение статистики, повторим это большое количество раз. По полученным значениям можно оценить функцию распределения. Функция распределения сама по себе для принятия решения нам не нужна, нам нужно p-value. Сравнив p-value с уровнем значимости, решаем, отклонять нулевую гипотезу или нет.

**P-value** — это вероятность получить заданное значение статистики или более экстремальное при верности нулевой гипотезы. Значение p-value вычисляется по формуле:

![\text{p-value} = \left\{    \begin{aligned}     &\mathbb{P}(T \le t | H_0) &&,\text{для левосторонней гипотезы} \\     &\mathbb{P}(T \ge t | H_0) &&,\text{для правосторонней гипотезы} \\     &2\times\min\left(\mathbb{P}(T \le t | H_0),\ \mathbb{P}(T \ge t | H_0)\right) &&,\text{для двусторонней гипотезы} \\    \end{aligned} \right.](https://habrastorage.org/getpro/habr/upload_files/10e/1b6/ef9/10e1b6ef9993eb9ddda2301bb23b543c.svg)

Чтобы оценить значение p-value двусторонней гипотезы, для реализации статистики t вычислим долю значений статистики из синтетических А/А тестов, которые оказались не больше t, и долю значений статистики из синтетических А/А тестов, которые оказались не меньше t. Возьмём меньшую из долей и умножим на два.

Реализуем описанный подход в коде. Допустим, данные имеют логнормальное распределение. Хотим проверить гипотезу о равенстве средних на группах размера 10. Есть 1 миллион исторических данных. Оценим распределение статистики по историческим данным с помощью 100 000 синтетических А/А тестов.

Код

```

```

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/3df/894/2f0/3df8942f052004b49459567462c8f488.png)

Кроме оценки плотности распределения статистики мы также построили критическую область критерия для ![\alpha=0.05](https://habrastorage.org/getpro/habr/upload_files/29d/baa/3e8/29dbaa3e86de13061bbd962041c595df.svg). Если статистика, рассчитанная по данным эксперимента, окажется в критической области, то отклоняем нулевую гипотезу о равенстве средних, иначе не отклоняем.

Проверим корректность полученного критерия. Проведём симуляцию 10 000 экспериментов с эффектом и без эффекта, построим распределения p-value.

Код

```

```

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/ee5/852/fd2/ee5852fd251320fe15c05ad9804877d2.png)

Критерий работает корректно, p-value на А/А тестах распределено равномерно, а на А/Б тестах выпукло вверх. 

Описанный подход построения критерия является весьма универсальным. Его можно использовать не только для проверки гипотезы о равенстве средних, но и для проверки других гипотез. Нужно выбрать подходящую статистику критерия и оценить её распределение по историческим данным при верности нулевой гипотезы.

Основным минусом подхода является необходимость иметь достаточно большое количество актуальных данных. Чем больше особенностей у распределения, тем больше потребуется исторических данных для построения корректного критерия. Исторические данные должны быть актуальные, то есть у них должно быть то же распределение, что и сейчас. Данные десятилетней давности, возможно, не подойдут.

Оценка распределения через синтетические А/А тесты – вычислительно затратная процедура. Если для вашей задачи классические критерии работают корректно и быстро, то разумно применять их.

## Итоги

Если обычные критерии работают некорректно, можно построить свой критерий. Этот критерий будет заточен под определённую статистику и природу данных. Плюсы: работает для любых гипотез и данных. Минусы: нужны актуальные исторические данные, вычислительно затратно.

Алгоритм построения и применения критерия:

1. Выберите статистику критерия, подходящую для проверяемой гипотезы.
    
2. Сгенерируйте множество значений статистики с помощью синтетические А/А тестов на исторических данных.
    
3. Вычислите значение статистики на данных эксперимента.
    
4. Оцените значение p-value для реализации статистики из п. 3 по множеству значений из п. 2.