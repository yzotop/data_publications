---
tags:
  - data
link: https://habr.com/ru/companies/ods/articles/325416/
source: habr
data_type:
  - AB tests
company: ODS
author: ODS
---
Здравствуйте, коллеги. Рассмотрим обычный онлайн-эксперимент в некоторой компании «Усы и когти». У неё есть веб-сайт, на котором есть красная кнопка в форме прямоугольника с закругленными краями. Если пользователь нажимает на эту кнопку, то где-то в мире мурлычет от радости один котенок. Задача компании — максимизация мурлыкания. Также есть отдел маркетинга, который усердно исследует формы кнопок и то, как они влияют на конверсию показов в клико-мурлыкания. Потратив почти весь бюджет компании на уникальные исследования, отдел маркетинга разделился на четыре противоборствующие группировоки. У каждой группировки есть своя гениальная идея того, как должна выглядеть кнопка. В целом никто не против формы кнопки, но красный цвет раздражает всех маркетологов, и в итоге было предложено четыре альтернативных варианта. На самом деле, даже не так важно, какие именно это варианты, нас интересует тот вариант, который максимизирует мурлыкания. Маркетинг предлагает провести A/B/n-тест, но мы не согласны: и так на эти сомнительные исследования спущено денег немерено. Попробуем осчастливить как можно больше котят и сэкономить на трафике. Для оптимизации трафика, пущенного на тесты, мы будем использовать шайку многоруких байесовских бандитов (bayesian multi-armed bandits). Вперед.

  

# A/B/n-тест

  

Будем считать, что клик — это некоторая случайная переменная ![$k$](https://habrastorage.org/getpro/habr/post_images/839/452/3d1/8394523d1f4bab66a8544cd365388d87.svg), принимающая значения ![$1$](https://habrastorage.org/getpro/habr/post_images/072/ceb/bbc/072cebbbc068c9c66ce5e423466d257c.svg) или ![$0$](https://habrastorage.org/getpro/habr/post_images/0fc/031/d00/0fc031d0014dcf57b36106bd16b8e71b.svg) с вероятностями ![$\theta$](https://habrastorage.org/getpro/habr/post_images/a21/ed4/61f/a21ed461f66aaee7817cacfb1a898860.svg) и ![$1 - \theta$](https://habrastorage.org/getpro/habr/post_images/352/d19/ecb/352d19ecb8459f5a80c17d658eec3e51.svg) соответственно. Такая величина имеет распределение Бернулли с параметром ![$\theta$](https://habrastorage.org/getpro/habr/post_images/a21/ed4/61f/a21ed461f66aaee7817cacfb1a898860.svg):

  

![$\large \begin{array}{rcl} k &\sim& \text{Bernoulli}\left(\theta\right) \\ p\left(k\right) &=& \theta^k \left(1 - \theta\right)^{1 - k} \end{array} $](https://habrastorage.org/getpro/habr/post_images/c26/581/471/c26581471b134a4d3bbacf3eb456003f.svg)

  

Вспомним, что среднее значение распределения Бернулли равно ![$\mu = \theta$](https://habrastorage.org/getpro/habr/post_images/2ed/b93/27c/2edb9327cd7eaf86864c06adb4423582.svg), а дисперсия равна ![$\sigma^2 = \theta\left(1 - \theta\right)$](https://habrastorage.org/getpro/habr/post_images/03d/4a7/4c5/03d4a74c52f17f33877978a8b942cc09.svg).

  

Попробуем для начала решить проблему с помощью обычного A/B/n-теста, n здесь означает, что тестируются не две гипотезы, а несколько. В нашем случае это пять гипотез. Но мы рассмотрим сначала ситуацию тестирования старого решения против нового, а затем обобщим на все пять случаев. В бинарном случае у нас есть две гипотезы:

  

- нулевая гипотеза ![$H_0: \theta_c = \theta_t$](https://habrastorage.org/getpro/habr/post_images/67a/f9c/ccd/67af9cccd65ecc3516b0337942de4267.svg) заключается в том, что нет никакой разницы в вероятности клика по старой кнопке ![$\theta_c$](https://habrastorage.org/getpro/habr/post_images/733/df0/7e7/733df07e7faf8fb5072e84c4b82213f1.svg) или по новой ![$\theta_t$](https://habrastorage.org/getpro/habr/post_images/f3f/cb7/d34/f3fcb7d3449503894ab2f25ecf2e325d.svg);
- альтернативная гипотеза ![$H_1: \theta_c < \theta_t$](https://habrastorage.org/getpro/habr/post_images/dd3/56d/9b6/dd356d9b68ef85f74b002d07a90c26ad.svg) заключается в том, что вероятность клика по старой кнопке ![$\theta_c$](https://habrastorage.org/getpro/habr/post_images/733/df0/7e7/733df07e7faf8fb5072e84c4b82213f1.svg) меньше чем по новой ![$\theta_t$](https://habrastorage.org/getpro/habr/post_images/f3f/cb7/d34/f3fcb7d3449503894ab2f25ecf2e325d.svg);
- вероятности ![$\theta_c$](https://habrastorage.org/getpro/habr/post_images/733/df0/7e7/733df07e7faf8fb5072e84c4b82213f1.svg) и ![$\theta_t$](https://habrastorage.org/getpro/habr/post_images/f3f/cb7/d34/f3fcb7d3449503894ab2f25ecf2e325d.svg) мы оцениваем как отношение кликов к показам в контрольной группе (control) и в экспериментальной/тестовой группе (treatment) соответственно.

  

Мы не можем знать истинное значение конверсии ![$\theta_c$](https://habrastorage.org/getpro/habr/post_images/733/df0/7e7/733df07e7faf8fb5072e84c4b82213f1.svg) на текущей вариации кнопки (на красной), но мы можем его оценить. Для этого у нас есть два механизма, которые работают сообща. Во-первых, это [закон больших чисел](https://ru.wikipedia.org/wiki/%D0%97%D0%B0%D0%BA%D0%BE%D0%BD_%D0%B1%D0%BE%D0%BB%D1%8C%D1%88%D0%B8%D1%85_%D1%87%D0%B8%D1%81%D0%B5%D0%BB), который утверждает, что какое бы ни было распределение случайной величины, если мы посемплируем достаточное количество примеров и усредним, то такая оценка будет близка к истинному значению среднего значения распределения (опустим пока индекс ![$c$](https://habrastorage.org/getpro/habr/post_images/b58/414/e96/b58414e960548906adae5a204bed502e.svg) для ясности):

  

![$\large \begin{array}{rcl} \overline{\theta}_n &=& \frac{1}{n}\sum_{i=1}^n k_i \\ \forall \epsilon \in \mathbb{R}: \lim_{n \rightarrow \infty} &=& P\left(\left|\overline{\theta}_n - \mu\right| < \epsilon\right) = 1 \end{array} $](https://habrastorage.org/getpro/habr/post_images/cbc/fb9/21a/cbcfb921af2dbaa0d2cb175e13462fb6.svg)

  

Во-вторых, это [центральная предельная теорема](https://ru.wikipedia.org/wiki/%D0%A6%D0%B5%D0%BD%D1%82%D1%80%D0%B0%D0%BB%D1%8C%D0%BD%D0%B0%D1%8F_%D0%BF%D1%80%D0%B5%D0%B4%D0%B5%D0%BB%D1%8C%D0%BD%D0%B0%D1%8F_%D1%82%D0%B5%D0%BE%D1%80%D0%B5%D0%BC%D0%B0), которая утверждает следующее: допустим, есть бесконечный ряд независимых и одинаково распределенных случайных величин ![$k_1, k_2, \ldots$](https://habrastorage.org/getpro/habr/post_images/153/b38/80a/153b3880a5934456fb3d3f122c50790b.svg), с истинным матожиданием ![$\mu$](https://habrastorage.org/getpro/habr/post_images/6af/297/d97/6af297d97b73ffa5591a8c2c5a6b6d86.svg) и дисперсией ![$\sigma^2$](https://habrastorage.org/getpro/habr/post_images/be0/006/9bd/be00069bd90f3669a29f6a12ff0df445.svg). Обозначим конечную сумму как ![$S_n = \sum_{i}^n k_i$](https://habrastorage.org/getpro/habr/post_images/8eb/c1b/feb/8ebc1bfeb3c49de609eab8cdbe50495d.svg), тогда

  

![$\large \frac{S_n - \mu n}{\sigma\sqrt{n}} \rightarrow \mathcal{N}\left(0, 1\right)\text{ при }n \rightarrow \infty $](https://habrastorage.org/getpro/habr/post_images/0fd/b22/efd/0fdb22efd77bfe5179527726e32ecb22.svg)

  

или эквивалентно тому, что при достаточно больших выборках оценка среднего значения имеет нормальное распределение с центром в ![$\mu$](https://habrastorage.org/getpro/habr/post_images/6af/297/d97/6af297d97b73ffa5591a8c2c5a6b6d86.svg) и дисперсией ![$\frac{\sigma^2}{n}$](https://habrastorage.org/getpro/habr/post_images/3bd/023/efa/3bd023efa02c83aeef2787748d280315.svg):

  

![$\large \overline{\theta} \sim \mathcal{N}\left(\mu, \frac{\sigma^2}{n}\right) $](https://habrastorage.org/getpro/habr/post_images/aa2/442/82d/aa244282d2f334252617c7c60fea6461.svg)

  

Казалось бы, бери да запускай два теста: раздели трафик на две части, жди пару дней, и сравнивай средние значения кликов по первому механизму. Второй механизм позволяет нам применять [t-тест](https://ru.wikipedia.org/wiki/T-%D0%9A%D1%80%D0%B8%D1%82%D0%B5%D1%80%D0%B8%D0%B9_%D0%A1%D1%82%D1%8C%D1%8E%D0%B4%D0%B5%D0%BD%D1%82%D0%B0) для оценки статистической значимости разницы средних значений выборок (потому что оценки средних имеют нормальное распределение). А также, увеличивая размер выборки ![$n$](https://habrastorage.org/getpro/habr/post_images/17e/d16/be1/17ed16be17023e4cd19584781c333593.svg), мы уменьшаем дисперсию оценки среднего, тем самым увеличивая свою уверенность. Но остается один важный вопрос: а какое количество трафика (пользователей) пустить на каждую вариацию кнопки? Статистика нам говорит, что если разница не нулевая, то это еще совсем не значит, что значения не равны. Возможно, просто не повезло, и первые ![$70\%$](https://habrastorage.org/getpro/habr/post_images/d11/3cd/995/d113cd995349f618fccb6a5220a7b441.svg) пользователей люто ненавидели какой-то цвет, и нужно подождать еще. И возникает вопрос, а сколько пользователей пустить на каждую вариацию кнопки, чтобы быть уверенным, что если разница и есть, то она существенная. Для этого нам придется еще раз встретиться с отделом маркетинга и задать им несколько вопросов, и, вероятно, нам придется им объяснить, что такое ошибки первого и второго рода. В общем, это может быть чуть ли не самым сложным во всем тесте. Итак, вспомним, что это за ошибки, и определим вопросы, которые следует задать отделу маркетинга, чтобы оценить количество трафика на вариацию. В конечном счете именно маркетинг принимает окончательно решение о том, какой цвет кнопки оставить.

  

|   |   |   |   |
|---|---|---|---|
||   |Верная гипотеза|   |
|![$H_0$](https://habrastorage.org/getpro/habr/post_images/58b/469/032/58b469032be3b2be41034475944fd45a.svg)|![$H_1$](https://habrastorage.org/getpro/habr/post_images/aca/400/f2d/aca400f2da98b443c10687ad0da2c95b.svg)|
|Ответ теста|![$H_0$](https://habrastorage.org/getpro/habr/post_images/58b/469/032/58b469032be3b2be41034475944fd45a.svg)|![$H_0$](https://habrastorage.org/getpro/habr/post_images/58b/469/032/58b469032be3b2be41034475944fd45a.svg) принята|![$H_0$](https://habrastorage.org/getpro/habr/post_images/58b/469/032/58b469032be3b2be41034475944fd45a.svg) неверно принята  <br>(ошибка II рода)|
|![$H_1$](https://habrastorage.org/getpro/habr/post_images/aca/400/f2d/aca400f2da98b443c10687ad0da2c95b.svg)|![$H_0$](https://habrastorage.org/getpro/habr/post_images/58b/469/032/58b469032be3b2be41034475944fd45a.svg) неверно отвергнута  <br>(ошибка I рода)|![$H_0$](https://habrastorage.org/getpro/habr/post_images/58b/469/032/58b469032be3b2be41034475944fd45a.svg) отвергнута|

  

Введем еще несколько понятий. Обозначим вероятность ошибки первого рода как ![$\alpha = P\left(H_1 \mid H_0\right)$](https://habrastorage.org/getpro/habr/post_images/91a/c56/7ad/91ac567ad689d6301c495f100539e6e3.svg), т.е. вероятность принятия альтернативной гипотезы при условии, что на самом деле верна нулевая гипотеза. Эта величина называется статистической значимостью (statistical significance). Также нам понадобится ввести вероятность ошибки второго рода ![$\beta = P\left(H_0 \mid H_1\right)$](https://habrastorage.org/getpro/habr/post_images/f17/5e6/e0d/f175e6e0d494d54897e809e40f0c2227.svg). Величина ![$1 - \beta$](https://habrastorage.org/getpro/habr/post_images/549/b8e/ab7/549b8eab724b1b8d035510be3cc5e854.svg) называется статистической мощностью (statistical power).

  

Рассмотрим изображение ниже, чтобы проиллюстрировать смысл статистической значимости и мощности. Предположим, что среднее значение распределения случайной величины на контрольной группе равно ![$\mu_c$](https://habrastorage.org/getpro/habr/post_images/cb6/62e/c5f/cb662ec5ff644d9b7ab6c794424f515d.svg), а на тестовой ![$\mu_t$](https://habrastorage.org/getpro/habr/post_images/d00/b53/706/d00b5370605f673e4daaea79e8b28ac3.svg). Выберем некоторый критический порог выше которого гипотеза ![$H_0$](https://habrastorage.org/getpro/habr/post_images/58b/469/032/58b469032be3b2be41034475944fd45a.svg) отвергается (вертикальная зеленая линия). Тогда если значение было правее порога, но из распределения контрольной группы, то мы его ошибочно припишем к распределению тестовой группы. Это будет ошибка первого рода, а значение ![$\alpha$](https://habrastorage.org/getpro/habr/post_images/8bb/e73/0e9/8bbe730e90adaf07435b47deee6a1b38.svg) — это площадь, закрашенная красным цветом. Соответственно, площадь области, закрашенной синим цветом, равна ![$\beta$](https://habrastorage.org/getpro/habr/post_images/f39/05b/5cf/f3905b5cfab08d98b2d380d5ea75c66c.svg). Величина ![$1 - \beta$](https://habrastorage.org/getpro/habr/post_images/549/b8e/ab7/549b8eab724b1b8d035510be3cc5e854.svg), которую мы назвали статистической мощностью, по сути показывает, сколько значений, соответствующих альтернативной гипотезе, мы действительно отнесем к альтернативной гипотезе. Проиллюстрируем вышеописанное.

  

```
# средние значение двух распределенийmu_c = 0.1mu_t = 0.6# дисперсии двух распределенийs_c = 0.25s_t = 0.3# порог: наблюдая значение исследуемой величины выше этого порога,# мы относим ее к синему распределениюc = 1.3*(mu_c + mu_t)/2
```

  

**Отрисовка графика**

  
![](https://habrastorage.org/r/w1560/files/475/9e5/ebc/4759e5ebcfc54b11a852704017d2d8ac.png)  

Обозначим этот порог буквой ![$c$](https://habrastorage.org/getpro/habr/post_images/b58/414/e96/b58414e960548906adae5a204bed502e.svg). Справедливо следующее равенство при условии, что случайная величина имеет нормальное распределение:

  

![$\large c = \mu + t\frac{\sigma}{\sqrt{n}} $](https://habrastorage.org/getpro/habr/post_images/daa/66b/f18/daa66bf18188f351f60b14ea9c4eaeab.svg)

  

где ![$t = P\left(X \leq t\right), X \sim \mathcal{N}\left(0, 1\right)$](https://habrastorage.org/getpro/habr/post_images/711/69c/9bc/71169c9bc77ff4aa13dbc8c2c0fbf91d.svg) — это [квантильная функция](https://ru.wikipedia.org/wiki/%D0%A4%D1%83%D0%BD%D0%BA%D1%86%D0%B8%D1%8F_%D0%BE%D1%88%D0%B8%D0%B1%D0%BE%D0%BA).

  

В нашем случае мы можем записать систему из двух уравнений для ![$\alpha$](https://habrastorage.org/getpro/habr/post_images/8bb/e73/0e9/8bbe730e90adaf07435b47deee6a1b38.svg) и ![$\beta$](https://habrastorage.org/getpro/habr/post_images/f39/05b/5cf/f3905b5cfab08d98b2d380d5ea75c66c.svg) соответственно:

  

![$\large\left\{ {c = \theta_c + t_{\alpha}\sqrt{\frac{\theta_c\left(1 - \theta_c\right)}{n}}\atop c = \theta_t + t_{\beta}\sqrt{\frac{\theta_t\left(1 - \theta_t\right)}{n}}} \right. $](https://habrastorage.org/getpro/habr/post_images/546/143/474/5461434746cd62f0cd35b1ef424d0fd8.svg)

  

Решив эту систему относительно ![$n$](https://habrastorage.org/getpro/habr/post_images/17e/d16/be1/17ed16be17023e4cd19584781c333593.svg), мы получим оценку достаточного количества трафика, который необходимо загнать в каждый эксперимент при заданной статистической значимости и мощности.

  

![$\large \begin{array}{rcl} \theta_c + t_{\alpha}\sqrt{\frac{\theta_c\left(1 - \theta_c\right)}{n}} &=& \theta_t + t_{\beta}\sqrt{\frac{\theta_t\left(1 - \theta_t\right)}{n}} \\ \theta_c \sqrt{n} + t_{\alpha}\sqrt{\theta_c\left(1 - \theta_c\right)} &=& \theta_t \sqrt{n} + t_{\beta}\sqrt{\theta_t\left(1 - \theta_t\right)} \\ n &=& \left(\frac{t_{\beta}\sqrt{\theta_t\left(1 - \theta_t\right)} - t_{\alpha}\sqrt{\theta_c\left(1 - \theta_c\right)}}{\theta_c - \theta_t}\right)^2 \end{array} $](https://habrastorage.org/getpro/habr/post_images/ecc/f7d/b11/eccf7db11534967cb49b300d8a9429dc.svg)

  

Например, если реальное значение конверсии ![$0.001$](https://habrastorage.org/getpro/habr/post_images/19b/617/3d1/19b6173d110982844e0d04e58d31d2ec.svg), а новое ![$0.0011$](https://habrastorage.org/getpro/habr/post_images/b7c/87f/215/b7c87f215d1608a29018a16ee76631cc.svg), тогда при ![$\alpha = \beta = 0.01$](https://habrastorage.org/getpro/habr/post_images/45c/0b7/c08/45c0b7c089a418707da519ef736dc890.svg) нам необходимо прогнать **2 269 319** пользователей через каждую вариацию. А новая вариация будет принята в случае, если ее конверсия превысит ![$0.00104$](https://habrastorage.org/getpro/habr/post_images/8d0/ef4/3d9/8d0ef43d9509feda1f768de8f2fd86a1.svg). Проверим это.

  

```
def get_size(theta_c, theta_t, alpha, beta):    # вычисляем квантили нормального распределения    t_alpha = stats.norm.ppf(1 - alpha, loc=0, scale=1)    t_beta = stats.norm.ppf(beta, loc=0, scale=1)    # решаем уравнение относительно n    n = t_alpha*np.sqrt(theta_t*(1 - theta_t))    n -= t_beta*np.sqrt(theta_c*(1 - theta_c))    n /= theta_c - theta_t    return int(np.ceil(n*n))n_max = get_size(0.001, 0.0011, 0.01, 0.01)print n_max# выводим порог, выше которого отклоняется H_0print 0.001 + stats.norm.ppf(1 - 0.01, loc=0, scale=1)*np.sqrt(0.001*(1 - 0.001)/n_max)>>>2269319>>>0.00104881009215
```

  

Таким образом, чтобы вычислить эффективный размер выборки, мы должны пойти в маркетинг или другой бизнес-департамент и узнать у них следующее:

  

- какое значение конверсии ![$\theta_t$](https://habrastorage.org/getpro/habr/post_images/f3f/cb7/d34/f3fcb7d3449503894ab2f25ecf2e325d.svg) на новой вариации должно получиться, чтобы бизнес принял решение перейти со старой версии кнопки с конверсией ![$\theta_c$](https://habrastorage.org/getpro/habr/post_images/733/df0/7e7/733df07e7faf8fb5072e84c4b82213f1.svg) на новую; естественно предположить, что если новое больше старого, то уже достаточно оснований для внедрения, но не стоит забывать про накладные расходы, как минимум на тестирование, ведь чем меньше разница, тем больше трафика нужно потратить на тест, а это потеря прибыли (при этом существует риск, что тест провалится);
- какой допустимый уровень значимости ![$\alpha$](https://habrastorage.org/getpro/habr/post_images/8bb/e73/0e9/8bbe730e90adaf07435b47deee6a1b38.svg) и мощности ![$\beta$](https://habrastorage.org/getpro/habr/post_images/f39/05b/5cf/f3905b5cfab08d98b2d380d5ea75c66c.svg) необходим (вероятно, тут еще придется как то объяснить бизнесу, что это значит).

  

Получив ![$n$](https://habrastorage.org/getpro/habr/post_images/17e/d16/be1/17ed16be17023e4cd19584781c333593.svg), мы можем легко вычислить порог ![$c$](https://habrastorage.org/getpro/habr/post_images/b58/414/e96/b58414e960548906adae5a204bed502e.svg). Это значит, что если мы соберем ![$n$](https://habrastorage.org/getpro/habr/post_images/17e/d16/be1/17ed16be17023e4cd19584781c333593.svg) наблюдений для каждой вариации, и окажется, что в экспериментальной группе оценка клика ![$\theta_t$](https://habrastorage.org/getpro/habr/post_images/f3f/cb7/d34/f3fcb7d3449503894ab2f25ecf2e325d.svg) выше порога ![$c$](https://habrastorage.org/getpro/habr/post_images/b58/414/e96/b58414e960548906adae5a204bed502e.svg), то мы отклоняем ![$H_0$](https://habrastorage.org/getpro/habr/post_images/58b/469/032/58b469032be3b2be41034475944fd45a.svg) и принимаем альтернативную гипотезу ![$H_1$](https://habrastorage.org/getpro/habr/post_images/aca/400/f2d/aca400f2da98b443c10687ad0da2c95b.svg) о том, что новая вариация лучше и вероятно мы ее внедрим. В этом случае у нас будет ![$\alpha \cdot 100\%$](https://habrastorage.org/getpro/habr/post_images/c2d/7bb/c01/c2d7bbc0197b2085c2d03311ec70b319.svg) шансов ошибки первого рода, т.е. мы ошибочно приняли решение заменить старую кнопку на новую. И ![$\left(1 - \beta\right)\cdot 100\%$](https://habrastorage.org/getpro/habr/post_images/546/007/77d/54600777df2cf36145c770ad232c380a.svg) шансов, что если новая вариация лучше, то мы правильно и сделали, что ее внедрили. Но самое печальное в том, что если после эксперимента, когда мы прогнали трафик через две вариации, окажется, что ![$\theta_t < c$](https://habrastorage.org/getpro/habr/post_images/b52/00d/413/b5200d413bc14a2e11511a09b1916dc3.svg), то мы считаем, что нет оснований отказываться от старой кнопки. Именно нет основания, мы не доказали, что новая кнопка хуже: мы как были в неопределенности, так и остались. Такие дела.

  

Честно говоря, мне кажется, что выводы, полученные в предыдущем абзаце, не очень убедительны, и вообще могут запутать не только вас. Представьте, что отдел маркетинга потратил пару миллионов рублей на уникальные исследования новой формы кнопки, а вы им такой «нет оснований считать, что она лучше». Кстати насчет, оснований: давайте проиллюстрируем, как решается уравнение относительно размера выборки и как изменяется распределение оценки среднего значения при увеличении размера выборки. На анимации вы можете наблюдать, как при увеличении размера выборки размах распределения оценки среднего уменьшается, т.е. уменьшается дисперсия оценки, которая и отождествляется с нашей уверенностью в этой оценке. На самом деле, изображение ниже немного нечестное: при уменьшении размаха распределения, оно еще и увеличивается в высоту, т.к. площадь должна оставаться постоянной.

  

**Рисуем анимацию**

```python
np.random.seed(1342)p_c = 0.3 p_t = 0.4alpha = 0.05beta = 0.2n_max = get_size(p_c, p_t, alpha, beta)c = p_c + stats.norm.ppf(1 - alpha, loc=0, scale=1)*np.sqrt(p_c*(1 - p_c)/n_max)print n_max, cdef plot_sample_size_frames(do_dorm, width, height):    left_x = c - width    right_x = c + width    n_list = range(5, n_max, 1) + [n_max]    for f in glob.glob("./../images/sample_size_gif/*_%s.*" %                        ('normed' if do_dorm else 'real')):        os.remove(f)    for n in tqdm_notebook(n_list):        s_c = np.sqrt(p_c*(1 - p_c)/n)        s_t = np.sqrt(p_t*(1 - p_t)/n)        c_c = p_c + stats.norm.ppf(1 - alpha, loc=0, scale=1)*s_c        c_t = p_t + stats.norm.ppf(beta, loc=0, scale=1)*s_t        support = np.linspace(left_x, right_x, 1000)        y_c = stats.norm.pdf(support, loc=p_c, scale=s_c)        y_t = stats.norm.pdf(support, loc=p_t, scale=s_t)        if do_dorm:            y_c /= max(y_c.max(), y_t.max())            y_t /= max(y_c.max(), y_t.max())        fig, ax = plt.subplots()        ax.plot(support, y_c, color='r', label='y control')        ax.plot(support, y_t, color='b', label='y treatment')        ax.set_ylim(0, height)        ax.set_xlim(left_x, right_x)        ax.axvline(c, color='g', label='c')        ax.axvline(c_c, color='m', label='c_c')        ax.axvline(c_t, color='c', label='c_p')        ax.axvline(p_c, color='r', alpha=0.3, linestyle='--', label='p_c')        ax.axvline(p_t, color='b', alpha=0.3, linestyle='--', label='p_t')        ax.fill_between(support[support <= c_t],                         y_t[support <= c_t],                         color='b', alpha=0.2, label='b: power')        ax.fill_between(support[support >= c_c],                         y_c[support >= c_c],                         color='r', alpha=0.2, label='a: significance')        ax.legend(loc='upper right', prop={'size': 20})        ax.set_title('Sample size: %i' % n, fontsize=20)        fig.savefig('./../images/sample_size_gif/%i_%s.png' %                     (n, 'normed' if do_dorm else 'real'), dpi=80)        plt.close(fig)plot_sample_size_frames(do_dorm=True, width=0.5, height=1.1)plot_sample_size_frames(do_dorm = False, width=1, height=2.5)!convert -delay 5 $(for i in $(seq 5 1 142); do echo ./../images/sample_size_gif/${i}_normed.png; done) -loop 0 ./../images/sample_size_gif/sample_size_normed.giffor f in glob.glob("./../images/sample_size_gif/*_normed.png"):    os.remove(f)!convert -delay 5 $(for i in $(seq 5 1 142); do echo ./../images/sample_size_gif/${i}_real.png; done) -loop 0 ./../images/sample_size_gif/sample_size_real.giffor f in glob.glob("./../images/sample_size_gif/*_real.png"):    os.remove(f)
```

Читерская анимация:

  
![](https://habrastorage.org/files/7a7/f72/a9a/7a7f72a9aed346f0b055f234a3f87445.gif)  

Реальная анимация:

  
![](https://habrastorage.org/files/b11/594/eb2/b11594eb2fc744b7a19f3907f1e8ce27.gif)  

В завершении раздела про A/B/n-тесты поговорим про n. Мы же в самом начале говорили, что у нас помимо текущей вариации есть четыре альтернативные гипотезы. Вышеописанный дизайн эксперимента легко масштабируется на n вариаций, в нашем случае их всего 5. Не поменяется ровным счетом ничего, кроме того, что теперь мы трафик разделяем на 5 частей и исследуем отклонение хотя бы одной из них выше критического порога ![$c$](https://habrastorage.org/getpro/habr/post_images/b58/414/e96/b58414e960548906adae5a204bed502e.svg). Но мы то с вами точно знаем, что лучшим вариантом является только один вариант, а это значит, что мы направляем ![$n\frac{k - 1}{k} = n \frac{4}{5}$](https://habrastorage.org/getpro/habr/post_images/38e/996/233/38e996233b69b327dfc2a851bcb67088.svg) (где ![$k$](https://habrastorage.org/getpro/habr/post_images/839/452/3d1/8394523d1f4bab66a8544cd365388d87.svg) — количество вариаций) на заведомо плохие вариации, мы просто пока не знаем, какие из них — плохие. А каждый пользователь, направленный на плохую вариацию, приносит нам меньше прибыли, чем если бы он был направлен на хорошую вариацию. В следующем разделе мы рассмотрим альтернативную методику дизайна экспериментов, которая крайне эффективна именно в онлайн-тестировании.

  

Ну и чтобы окончательно вас добить сложностями проведения A/B/n-тестов, рассмотрим одну из возможных поправок значимости результата при множественном тестировании гипотез. Например, мы решили, что ![$\alpha = 0.05$](https://habrastorage.org/getpro/habr/post_images/114/307/e4b/114307e4bd883bf0bdde46d49a931b0d.svg), а тестов в эксперименте пять, тогда:

  

![$\large\begin{array}{rcl} P\left(\text{хотя бы один результат значимый}\right) &=& 1 - P\left(\text{все результаты незначимы}\right) \\ &=& 1 - \left(1 - 0.05\right)^5 \\ &=& 1 - 0.95^5 \\ &\approx& 0.2262 \end{array}$](https://habrastorage.org/getpro/habr/post_images/370/b7f/f4d/370b7ff4d7fa4e753dcf267cda065865.svg)

  

Получается, что если даже все тесты не значимы, то все равно ![$22.62\%$](https://habrastorage.org/getpro/habr/post_images/2b0/111/f23/2b0111f2389eaf8c3372b06df8fc81c1.svg) шансов, что мы ошибочно получим значимый результат. Неожиданно, правда? Попробуем разобраться. Допустим, у нас есть ![$n$](https://habrastorage.org/getpro/habr/post_images/17e/d16/be1/17ed16be17023e4cd19584781c333593.svg) тестов в рамках одного эксперимента. Для каждого заданы свои p-values ![$p_i$](https://habrastorage.org/getpro/habr/post_images/820/8f4/636/8208f46368a6c37bc01d6a81fa84d671.svg) (вероятность ошибочно отклонить ![$H_0$](https://habrastorage.org/getpro/habr/post_images/58b/469/032/58b469032be3b2be41034475944fd45a.svg), а это и есть наши ![$\alpha$](https://habrastorage.org/getpro/habr/post_images/8bb/e73/0e9/8bbe730e90adaf07435b47deee6a1b38.svg)). Что если мы уменьшим уровни значимости в ![$n$](https://habrastorage.org/getpro/habr/post_images/17e/d16/be1/17ed16be17023e4cd19584781c333593.svg) раз:

  

![$\large\begin{align} P \left({ \bigcup_{i=1}^{n}\left( p_i<\frac{\alpha}{n}\right) }\right) &\le \sum_{i=1}^{n}P\left(p_i<\frac{\alpha}{n}\right)\\ &\le \sum_{i=1}^{n}\frac{\alpha}{n}\\ &= n \frac{\alpha}{n}\\ &= \alpha \end{align}$](https://habrastorage.org/getpro/habr/post_images/a1b/aba/ac5/a1babaac5063e7d9c21fe3950446d214.svg)

  

Тогда вероятность того, что хотя бы один результат значим, будет наше исходное ![$\alpha = 0.05$](https://habrastorage.org/getpro/habr/post_images/114/307/e4b/114307e4bd883bf0bdde46d49a931b0d.svg). Проверим:

  

![$\large\begin{array}{rcl} P\left(\text{хотя бы один результат значимый}\right) &=& 1 - P\left(\text{все результаты незначимы}\right) \\ &=& 1 - \left(1 - 0.01\right)^5 \\ &=& 1 - 0.99^5 \\ &\approx& 0.0491 \end{array}$](https://habrastorage.org/getpro/habr/post_images/b87/c86/ac8/b87c86ac8d2bc22856ba9277a7b63a8f.svg)

  

Такой трюк называется [поправкой Холма-Бонферонни](https://ru.wikipedia.org/wiki/%D0%9A%D0%BE%D1%80%D1%80%D0%B5%D0%BA%D1%86%D0%B8%D1%8F_%D0%BD%D0%B0_%D0%BC%D0%BD%D0%BE%D0%B6%D0%B5%D1%81%D1%82%D0%B2%D0%B5%D0%BD%D0%BD%D0%BE%D0%B5_%D1%82%D0%B5%D1%81%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5). Таким образом, чем больше гипотез, тем меньше требуемый уровень значимости, тем больше пользователей нужно нагнать в каждый тест. Возвращаясь к предыдущему примеру, отметим, что нам было необходимо **2 269 319** пользователей, но с поправкой нужно **2 853 873** пользователей на каждую вариацию. А это, на минуточку, почти на ![$26\%$](https://habrastorage.org/getpro/habr/post_images/df3/e09/8fe/df3e098fe53ce8a817e5d8402d3548be.svg) больше трафика на весь эксперимент. Если углубиться во все возможные поправки, то получится на небольшую книжку по статистике. Такие дела.

  

**История про зомби-лосося**

  

# Байесовские многорукие бандиты

  

Сразу стоит отметить, бандиты не являются универсальной заменой A/B-тестированию, но являются отличной заменой в определенных областях. A/B-тестирование появилось больше века назад, и за все время применялось для тестирования гипотез в различных областях, таких как медицина, сельское хозяйство, экономика и т. д. У всех этих областей есть несколько общих факторов:

  

- стоимость одного эксперимента существенна (представьте какое-нибудь новое средство от страшной болезни: помимо того, чтобы его произвести, необходимо еще найти людей, которые согласятся принимать таблетку, не зная, является она лекарством или плацебо);
- ошибки обходятся крайне дорого (опять же, факт из медицины: если фарм-компания спустя 10 лет исследований выпустит на рынок средство, которое не работает, вероятно, этой фарм-компании конец, а также людям, которые ей доверились).

  

A/B-тестирование идеально подходит для классических экспериментов. Оно позволяет оценить размер выборки для теста, что позволяет также заранее оценить бюджет. А также оно позволяет задать необходимые уровни ошибки: скажем, медики могут поставить крайне небольшие ![$\alpha$](https://habrastorage.org/getpro/habr/post_images/8bb/e73/0e9/8bbe730e90adaf07435b47deee6a1b38.svg) и ![$\beta$](https://habrastorage.org/getpro/habr/post_images/f39/05b/5cf/f3905b5cfab08d98b2d380d5ea75c66c.svg), в то время как других не так волнует мощность, но волнует значимость. Например, при тестировании нового рецепта бургеров мы не хотим подавать новое, если оно реально хуже старого; но если мы ошиблись в другую сторону, т.е. новый рецепт лучше, но мы его не приняли, то это мы легко переживем, т.к. до сегодняшнего дня мы ведь как-то жили с этим и не разорились. Вопрос только упирается в бюджет, который мы готовы потратить на эксперимент, если он у нас бесконечен, то, конечно, нам проще приблизить оба порога к нулю и прогнать миллионы экспериментов.

  

Но в мире онлайн-экспериментов всё не так критично, как в классических экспериментах. Цена ошибки близка к нулю, цена эксперимента тоже близка к нулю. Да, это не исключает того, что можно применять A/B-тесты. Но есть вариант лучше, который не просто тестирует гипотезы и дает ответ, какая из них, возможно, лучше, а ещё и динамически изменяет свои оценки того, какая вариация лучше, и, наконец, принимает решение, сколько трафика послать на ту или иную вариацию в данный момент времени.

  

![](https://habrastorage.org/r/w1560/files/407/4f6/c96/4074f6c9670b406ca13b1f720415c7d1.jpg)

  

Вообще, это один из многих компромиссов, с которым приходится жить в машинном обучении: в данном случае — это исследование других вариантов против эксплуатации лучшего (exploration vs exploitaion trade-off). В процессе тестирования двух или более гипотез (вариаций кнопки) мы не хотим посылать на заведомо неверные варианты большое количество трафика (в случае A/B-теста мы посылаем равные доли на каждую вариацию). Но в то же время, мы хотим следить и за остальными вариациями, и в случае, если нам вначале не повезло, или сменилась мода на цвета кнопок, мы бы могли почувствовать это и исправить ситуацию. На помощь к нам приходят байесовские бандиты. Вот они.

  

![](https://habrastorage.org/r/w1560/files/5cf/d77/762/5cfd77762bc3436a84074095b73abae0.jpg)

  

Описанный дизайн очень напоминает ситуацию, когда вы приходите в казино, и перед вами стоит ряд игральных автоматов типа «однорукий бандит». У вас ограниченное количество денег и времени, и вы хотели бы по-быстрому найти лучший автомат, при этом понеся как можно меньше расходов. Такая постановка задачи называется задачей о многоруком бандите. Существует множество подходов к этой задаче, начиная от простой ![$\epsilon$](https://habrastorage.org/getpro/habr/post_images/e12/dff/45a/e12dff45a7ed01bc64ba2aeb4d34a2bc.svg)-жадной стратегии, когда вы с вероятностью ![$\epsilon$](https://habrastorage.org/getpro/habr/post_images/e12/dff/45a/e12dff45a7ed01bc64ba2aeb4d34a2bc.svg) дергаете за ту ручку, которая принесла вам больше всего прибыли к текущему моменту, и с вероятностью ![$1 - \epsilon$](https://habrastorage.org/getpro/habr/post_images/c7a/09d/4bd/c7a09d4bdbada5d76e8a7afae6e2b4e1.svg) вы дергаете случайную руку (имеется в виду рычаг игрового автомата). Есть более сложные способы, основанные на анализе потерь при игре неоптимальной рукой. Но мы рассмотрим другой способ, основанный на [семплировании Томпсона](https://en.wikipedia.org/wiki/Thompson_sampling) и теореме Байеса. Способ не очень новый (1933 год), но он стал востребован только в эпоху онлайн-экспериментов. Yahoo была одной из первых компаний, которые стали использовать этот метод для персонализации новостной выдачи в начале 2010-ых годов. Позже Microsoft стал использовать бандитов для оптимизации CTR-баннеров в выдаче поиска Bing. В принципе, основная масса статей про современные онлайн-исследования пишется ими. Кстати, постановка задачи очень напоминает обучение с подкреплением, за одним лишь исключением: в постановке задачи с бандитами агент не может изменять окружение, а в общем обучении с подкреплением агент может влиять на среду.

  

Формализуем бандитскую задачу. Пусть к моменту времени ![$t$](https://habrastorage.org/getpro/habr/post_images/9b6/547/e8d/9b6547e8d3f9c2d6f92fbc781ac55660.svg) мы наблюдаем последовательность наград ![$\vec{y}_t = \left(y_1, y_2, \ldots, y_t\right)$](https://habrastorage.org/getpro/habr/post_images/895/c76/36a/895c7636ac0f65f445b0a74a91c59329.svg). Обозначим действие, принятое в момент времени ![$t$](https://habrastorage.org/getpro/habr/post_images/9b6/547/e8d/9b6547e8d3f9c2d6f92fbc781ac55660.svg) как ![$a_t$](https://habrastorage.org/getpro/habr/post_images/6e8/2bf/ada/6e82bfada1628f966e656803f44dc211.svg) (индекс бандита, цвет кнопки). Также считаем, что каждый ![$y_t$](https://habrastorage.org/getpro/habr/post_images/87f/0ea/369/87f0ea3697ba1c1981db5c1eec3d0831.svg) сгенерирован независимо из некоторого распределения наград своего бандита ![$f_{a_t}\left(y \mid \vec{\theta}\right)$](https://habrastorage.org/getpro/habr/post_images/c05/419/27e/c0541927eb58ccdbcf675ade5c0f5b0d.svg), где ![$\vec{\theta}$](https://habrastorage.org/getpro/habr/post_images/6cd/b1e/9d4/6cdb1e9d4cb25f5d9a5f005ed74e1837.svg) — это некоторый вектор параметров. В случае анализа кликов задача упрощается тем, что награда является бинарной ![$y_t \in \left\{0, 1\right\}$](https://habrastorage.org/getpro/habr/post_images/18c/791/0c7/18c7910c70c40c9fa8fc73a2dc0cd291.svg), а вектор параметров — это просто параметры распределения Бернулли каждой вариации.

  

Тогда ожидаемой наградой бандита ![$a$](https://habrastorage.org/getpro/habr/post_images/366/a4b/5c5/366a4b5c5bdf260210b49a89f904efb9.svg) будет ![$\mu_a\left(\vec{\theta}\right) = \mathbb{E}\left[y_t \mid \vec{\theta}, a_t = a\right]$](https://habrastorage.org/getpro/habr/post_images/13f/8fc/0f5/13f8fc0f584396878e9a8c4b52a6593f.svg). Если бы параметры распределений были известны, то мы легко бы вычислили ожидаемые награды и просто выбирали бы всегда вариант с максимальной наградой. Но, к сожалению, истинные параметры распределений нам неизвестны. Также обратим внимание, что в случае с кликами ![$f_a\left(y \mid \theta_a\right) = \theta_a^y \left(1 - \theta_a\right)^{1 - y}$](https://habrastorage.org/getpro/habr/post_images/aac/6fa/ab1/aac6faab1dc0e098f624bfc67d2ff567.svg), а ожидаемая награда ![$\mu_a = \theta_a$](https://habrastorage.org/getpro/habr/post_images/00b/16d/008/00b16d00869b4a545391df2dce136027.svg). В таком случае мы можем ввести априорное распределение на параметры распределения Бернулли и после каждого действия, наблюдая награду, сможем обновлять нашу «веру» в бандита ![$a$](https://habrastorage.org/getpro/habr/post_images/366/a4b/5c5/366a4b5c5bdf260210b49a89f904efb9.svg), используя теорему Байеса. Для этого идеально подходит [бета распределение](https://ru.wikipedia.org/wiki/%D0%91%D0%B5%D1%82%D0%B0-%D1%80%D0%B0%D1%81%D0%BF%D1%80%D0%B5%D0%B4%D0%B5%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5) по трем причинам:

  

- бета распределение является [априорно сопряженным](https://ru.wikipedia.org/wiki/%D0%A1%D0%BE%D0%BF%D1%80%D1%8F%D0%B6%D1%91%D0%BD%D0%BD%D0%BE%D0%B5_%D0%B0%D0%BF%D1%80%D0%B8%D0%BE%D1%80%D0%BD%D0%BE%D0%B5_%D1%80%D0%B0%D1%81%D0%BF%D1%80%D0%B5%D0%B4%D0%B5%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5) к распределению Бернулли, т.е. апостериорное распределение тоже имеет форму бета-распределения, но с другими параметрами:

  

![$\large\begin{array}{lcr} p\left(\theta \mid y\right) &\propto& p\left(\theta\right) \cdot p\left(y \mid \theta\right) \\ &\propto& \frac{1}{\text{B}\left(\alpha, \beta\right)} \theta^{\alpha - 1} \left(1 - \theta\right)^{\beta - 1} \cdot \theta^y \left(1 - \theta\right)^{1 - y} \\ &\propto& \theta^{\alpha - 1 + y} \left(1 - \theta\right)^{\beta - 1 + 1 - y} \end{array}$](https://habrastorage.org/getpro/habr/post_images/517/25b/a3d/51725ba3d8b7538c1375da12220d275d.svg)

  

![$\large\begin{array}{lcr} \text{Beta}\left(\alpha + y, \beta + 1 - y\right) &=& \text{Beta}\left(\theta \mid \alpha, \beta\right) \cdot \text{Bernoulli}\left(y \mid \theta \right) \end{array}$](https://habrastorage.org/getpro/habr/post_images/ad4/2ff/d98/ad42ffd98cda18c0301211e3f9174e1b.svg)

  

- при ![$\alpha = \beta = 1$](https://habrastorage.org/getpro/habr/post_images/0ee/ed9/72c/0eeed972c3cb7f49e4648770e671a6d4.svg) бета-распределение принимает форму равномерного распределения, т.е. именно такого, которое естественно использовать в ситуации полной неопределенности (например, в самом начале тестирования); чем более определенными становятся наши ожидания относительно прибыльности бандита, тем более скошенным становится распределение (влево — не прибыльный бандит, вправо — прибыльный);
- модель легко интерпретируема, ![$\alpha$](https://habrastorage.org/getpro/habr/post_images/8bb/e73/0e9/8bbe730e90adaf07435b47deee6a1b38.svg) — это количество успешных испытаний, а ![$\beta$](https://habrastorage.org/getpro/habr/post_images/f39/05b/5cf/f3905b5cfab08d98b2d380d5ea75c66c.svg) — количество неуспешных испытаний; среднее значение будет ![$\frac{\alpha}{\alpha + \beta}$](https://habrastorage.org/getpro/habr/post_images/4a4/194/3ef/4a41943effcfd954b128d2cdf73393f9.svg).

  

Посмотрим функции плотности для различных параметров бета-распределения:

  

![$\large\begin{array}{lcr} f\left(\theta, \alpha, \beta\right) &=& \frac{1}{\text{B}\left(\alpha, \beta\right)} \theta^{\alpha - 1} \cdot \left(1 - \theta\right)^{\beta - 1} \end{array}$](https://habrastorage.org/getpro/habr/post_images/b51/e55/481/b51e55481be69dccf0acaf7d2d2b6677.svg)

  

```
support = np.linspace(0, 1, 1000)fig, ax = plt.subplots()for a, b in [(1, 1), (5, 5), (5, 10), (5, 25), (1, 7), (7, 1)]:    ax.plot(support, stats.beta.pdf(support, a, b),             label='a=%i, b=%i' % (a, b))ax.legend(loc='upper right', prop={'size': 20})ax.set_title('Beta PDFs', fontsize=20)
```

  
![](https://habrastorage.org/r/w1560/files/7a6/b8a/b59/7a6b8ab59f7d451dbe52735825c8ba6a.png)  

Получается, что имея некоторые априорные ожидания об истинном значении конверсии, мы можем использовать теорему Байеса и обновлять наши ожидания при поступлении новых данных. А форма априорного распределения после обновления ожиданий остается все тем же бета-распределением. Итак, у нас есть следующая модель:

  

![$\large\begin{array}{lcr} \theta_i &\sim& \text{Beta}\left(\alpha_i, \beta_i\right) \\ y_i &\sim& \text{Bernoulli}\left(\theta_i\right) \end{array}$](https://habrastorage.org/getpro/habr/post_images/5b5/6ee/f6c/5b56eef6c5eb0a7890e1aa02bda7fff4.svg)

  

Тут мы применяем вторую, вполне естественную эвристику: давайте будем для нового пользователя (новый эксперимент) семплировать руку бандита (вариацию кнопки) не случайно, а пропорционально нашим текущим ожиданиям о полезности этой руки. Но так как у нас байесовская постановка задачи, и мы не храним в явном виде наши оценки полезности, а храним распределение оценок полезности для каждой руки, то и алгоритм семплирования немного изменяется:

  

- Для всех бандитов введем два параметра бета-распределения и приравняем их к единице ![$\forall i, \alpha_i = \beta_i = 1$](https://habrastorage.org/getpro/habr/post_images/7b2/6be/0e7/7b26be0e7482aa1a02742a9f592296f7.svg);
- повторяем в течении некоторого времени ![$t = 1, 2, \ldots$](https://habrastorage.org/getpro/habr/post_images/f3e/ac4/ce8/f3eac4ce88da49479706d5ac70a1efba.svg)  
    - для каждого бандита семплируем ![$\theta_i \sim \text{Beta}\left(\alpha_i, \beta_i\right)$](https://habrastorage.org/getpro/habr/post_images/425/936/12b/42593612b4f6171db903168419e9b3f5.svg);
    - выбираем бандита с максимальной наградой ![$k = \arg\max_{i} \theta_i$](https://habrastorage.org/getpro/habr/post_images/32b/b39/fb1/32bb39fb1745d198b87baff7000ff286.svg);
    - используем ![$k$](https://habrastorage.org/getpro/habr/post_images/839/452/3d1/8394523d1f4bab66a8544cd365388d87.svg)-ого бандита в текущем эксперименте и получаем награду ![$y \in \left\{0, 1\right\}$](https://habrastorage.org/getpro/habr/post_images/4e8/f00/4d5/4e8f004d513c4e1b910df9ff5ecce9cf.svg) (показываем ту кнопку текущему пользователю, которая по текущему семплу максимизирует награду);
    - обновляем параметры соответствующего априорного распределения (легко модифицируется для batch mode, если мы проводим не один, а серию экспериментов):
    - ![$\alpha_i := \alpha_i + y$](https://habrastorage.org/getpro/habr/post_images/8fe/f80/dc8/8fef80dc8d04afe1b55944ad2f330202.svg)  
        
    - ![$\beta_i := \beta_i + 1 - y$](https://habrastorage.org/getpro/habr/post_images/ed4/1d0/559/ed41d05598b6e3c84ccffba1106c7b19.svg)  
        

  

Семплирование Томпсона является эмпирикой, но для некоторых случаев доказаны теоремы асимптотической сходимости. В том числе и для Beta-Bernoulli случая, т.е. при ![$t \rightarrow \infty$](https://habrastorage.org/getpro/habr/post_images/98f/9d3/546/98f9d35466eddf699f1cce69f4de7cf2.svg) вероятность того, что бандиты с семплированием Томпсона сойдутся к верному решению, равна единице. Но это все теории, на практике нам важно следующее:

  

- если обозначить вероятность выигрыша на лучшем бандите за ![$\theta^*$](https://habrastorage.org/getpro/habr/post_images/845/41f/966/84541f96630c2bcb35f0bb0911a3fe8d.svg), то доля трафика, потраченного на эту опцию будет пропорциональна ![$\theta^*$](https://habrastorage.org/getpro/habr/post_images/845/41f/966/84541f96630c2bcb35f0bb0911a3fe8d.svg); в процессе обучения точность оценки будет возрастать, и доля трафика, потраченного на нужную опцию, будет расти;

  

![](https://habrastorage.org/r/w1560/files/9b7/854/690/9b78546906cd449fa4a060228d74a8a2.png)

  

- в любой момент времени у нас есть параметры распределения вероятностей выигрыша на каждой вариации (параметры бета-распределения), это позволяет нам применять всё те же тесты, что и при классическом тестировании;
- но отметим, что в отличие от тестов, мы не хотим получить ответ в виде статистической значимости разности кликов по разным вариациям, мы, как бизнес, хотим максимизировать CTR, а именно это и делает бандитская модель.

  

# Эксперимент

  

Проведем симуляционный эксперимент. Допустим, у нас всё те же пять вариаций кнопки, но со временем предпочтения пользователей нашего мурлычного сайта плавно меняются. В принципе, динамика уже сразу исключает применение одного A/B/n-теста, ну и ладушки.

  

```
# количество вариацийn_variations = 5# сколько раз меняются предпочтенияn_switches = 5# каждые n_period_len показов кнопки предпочтения пользователей меняютсяn_period_len = 2000# истинные вероятности клика по каждой вариации# напомню, что длина периода, в течение которого тренд не меняется, равна n_period_lenp_true_periods = np.array([        [1, 2, 3, 4, 5],        [2, 3, 4, 5, 1],        [3, 4, 5, 2, 1],        [4, 5, 3, 2, 1],        [5, 4, 3, 2, 1]], dtype=np.float32).T**3p_true_periods /= p_true_periods.sum(axis=0)# отрисуем истинные вероятностиax = sns.heatmap(p_true_periods)ax.set(xlabel='Periods', ylabel='Variations', title='Probabilities of variations')plt.show()
```

  
![](https://habrastorage.org/r/w1560/files/161/7b3/233/1617b32337b5446caa5bc6ed0bd250e2.png)  

Теперь запустим симуляцию: будем выводить после каждого периода наши оценки вероятности клика по каждой вариации и убедимся, что к концу каждого периода находится оптимальная вариация. Также будем следить за тем, сколько трафика из `n_period_len` отправлено на каждую вариацию.

  

В первый период мы четко определяем победителя (самый правый график с явно выраженным пиком). Напоминаю, что каждый график на изображении — это распределение наших ожиданий относительно качества руки (конверсии вариации). Чем шире график, тем больше неопределенности (прямая линия/равномерное распределение — это полная неопределенность); чем уже, тем меньше неопределенности. Но вот во второй период мы уже приходим не с полной неопределенностью (равномерное распределение на параметры), а со статистикой, собранной в первый период, и потому мы видим, что существенная доля трафика уходит на старого победителя. К следующим периодам новым лучшим вариациям все труднее выходить в лидеры: обратите внимание, что дисперсия бывших лидеров (ширина пика) меняется очень медленно. Потому часто алгоритм не успевает сойтись за заданное количество шагов. Существует множество способов исправления этой ситуации, самым простым является постепенное выравнивание распределения (приведение к равномерному, т.е. со временем мы теряем уверенность в прошлых оценках — это вполне естественно). Запустим симуляцию без затухания и посмотрим на результаты после каждого периода.

  

**Симуляция и отрисовка анимации**

  

Период первый:

  

```
Winner is 44    19533      192      121       80       8dtype: int64
```

  
![](https://habrastorage.org/r/w1560/files/4b5/9ee/aff/4b59eeaffa484ff38f045ff62f99ebca.png)  

Период второй:

  

```
Winner is 43    13974     5931       62       30       1dtype: int64
```

  
![](https://habrastorage.org/r/w1560/files/e70/881/7ad/e708817ad3d3457eb76c3fb932f0501e.png)  

Период третий:

  

```
Winner is 42    10643     6754     2551       30       3dtype: int64
```

  
![](https://habrastorage.org/r/w1560/files/16a/9de/61c/16a9de61cd9b4ef9b766c22612728951.png)  

Период четвертый:

  

```
Winner is 41    9832    6954    1443    1340     44dtype: int64
```

  
![](https://habrastorage.org/r/w1560/files/8e5/c00/5d9/8e5c005d98364f1e8fb5a6d5d77d5bab.png)  

Период пятый:

  

```
Winner is 41    11090     8884       22       1dtype: int64
```

  
![](https://habrastorage.org/r/w1560/files/23b/e06/58f/23be0658f0e74f3bb9b90c91a2d687e4.png)  

Отрисуем долю трафика, потраченного на каждую вариацию в каждый период.

  

```
df = []for pid in actionspp.keys():    for vid in actionspp[pid].keys():        df.append({                'Period': pid,                'Variation': vid,                'Probapility': actionspp[pid][vid]/1000.0            })df = pd.DataFrame(df)ax = sns.barplot(x="Period", y="Probapility", hue="Variation", data=df)ax.set(title="Ratio of traffic used for variations per period")plt.show()
```

  
![](https://habrastorage.org/r/w1560/files/927/e7c/fd7/927e7cfd7caf4cc696af42c075ec0dc9.png)  

Ну и под конец нарисуем в динамике изменения наших ожиданий относительно крутости каждой вариации.

  

**Симуляция и отрисовка анимации**

  

**Результат без применения затухания (тяжелая гифка, 40мб)**

  

**Результат с применением затухания (тяжелая гифка, 40мб)**

  

# Заключение

  

В заключение, отметим еще несколько преимуществ бандитов для онлайн-экспериментов:

  

- бандиты позволяют добавлять новые вариации прямо в процессе эксперимента;
- если у вас есть некоторые априорные ожидания о том, что новая вариация лучше остальных, то вы можете добавить ее в эксперимент, изменив ожидания с равномерного распределения ![$\alpha = \beta = 1$](https://habrastorage.org/getpro/habr/post_images/0ee/ed9/72c/0eeed972c3cb7f49e4648770e671a6d4.svg) на смещенное ожидание в сторону единицы ![$\alpha = 10, \beta = 5$](https://habrastorage.org/getpro/habr/post_images/520/4ec/71d/5204ec71d0dc5653c0afaa5eefe62acf.svg);
- в случае, если варианты разнесены по инфраструктуре, и один из сервисов вышел из строя, то бандитская модель быстро это заметит и перестанет лить трафик в сломанную руку.

  

![](https://habrastorage.org/r/w1560/files/27b/c9f/8c7/27bc9f8c7308481580a6960105721c06.jpg)

  

Самое главное: при использовании бандитов в разы увеличивается количество мурлыкающих котиков, разве это не прекрасно? Все остальные аргументы вообще ничего не стоят.

  

_Спасибо Насте [bauchgefuehl](https://habrahabr.ru/users/bauchgefuehl/), Арсению [arseny_info](https://habrahabr.ru/users/arseny_info/) и Юре [yorko](https://habrahabr.ru/users/yorko/) за редактирование и советы._