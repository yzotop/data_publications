---
tags:
  - data
link: https://www.linkedin.com/pulse/ratio-%2525D0%2525BC%2525D0%2525B5%2525D1%252582%2525D1%252580%2525D0%2525B8%2525D0%2525BA%2525D0%2525B8-%2525D0%2525B2-ab-%2525D1%252582%2525D0%2525B5%2525D1%252581%2525D1%252582%2525D0%2525B0%2525D1%252585-polina-egubova-6tedc/?trackingId=1Uh0Aag4Ri6vGRDhas9cMw%3D%3D
data_type:
  - AB tests
source: linkedin
---
**В user-метриках** в качестве объекта выступает пользователь, при этом все **наблюдения являются независимыми** (при условии отсутствия сетевых эффектов, конечно). Другими словами, информация об одном юзере никак не помогает нам узнать информацию о другом.

**В ratio-метриках** это не так, поскольку мы используем несколько значений для каждого объекта (то есть, пользователя), а они уже в свою очередь **являются зависимыми**: прошлое поведение юзера позволяет делать выводы о его будущем поведении, потому что это один и тот же человек.

**Зависимость данных не позволяет нам применить критерий Стьюдента** для оценки значимости наблюдаемой разницы ratio-метрик между двумя группами в А/В-тесте, поскольку он предполагает независимость наблюдений, так что его использование сильно растит ошибку первого рода.

Задачу можно решить с помощью **бутстрэпа**, но он довольно трудоемкий с точки зрения вычислительного времени, поэтому в большинстве случаев вместо него я использую дельта-метод.

**Проблема критерия Стьюдента** с точки зрения ratio-метрик в том, что он не учитывает зависимость данных, а потому **неправильно считает дисперсию**. Дельта-метод как раз решает эту проблему и позволяет корректно оценить дисперсию отношения двух случайных величин.

## Код на python для расчета разницы средних, p-value и построения доверительного интервала

Ниже - мой код на python, с помощью которого я ежедневно считаю ratio-метрики в A/B-тестах. Я дополнила его комментариями, так что он стал совсем понятным и простым. Здесь на входе мы имеем значения числителей и знаменателей метрики в контрольной и тестовой группах. На выходе - разницу средних и p-value, а также доверительный интервал (в абсолютном и относительном выражении).

```python
import numpy as np
from scipy import stats

# мы провели АВ-тест и собрали данные по ratio-метрике (числители и знаменатели) в контрольной и тестовой группах:

control_numerators, control_denominators
test_numerators, test_denominators

# рассчитаем дисперсию для ratio-метрики в контроле и тесте:

def get_ratio_metric_variance(numerators, denominators):
    n = len(numerators)

    numerators_mean = np.mean(numerators)
    denominators_mean = np.mean(denominators)

    ddof = 1 # delta degrees of freedom
    numerators_variance = np.var(numerators, ddof) 
    denominators_variance = np.var(denominators, ddof)

    covariance = np.cov(numerators, denominators, ddof)[0][1]

    ratio_metric_variance = (numerators_variance/numerators_mean**2 + denominators_variance/denominators_mean**2 - 2*covariance/(numerators_mean**2))*(numerators_mean**2)/(denominators_mean*denominators_mean*n)

    return ratio_metric_variance


control_variance = get_ratio_metric_variance(control_numerators, control_denominators)

test_variance = get_ratio_metric_variance(test_numerators, test_denominators)


control_mean = control_numerators.sum() / control_denominators.sum()
test_mean = test_numerators.sum() / test_denominators.sum()

mean_difference = test_mean - control_mean
mean_difference_relative = mean_difference * 100.0 / control_mean

variance = control_variance + test_variance

z = mean_difference / np.sqrt(variance)
p_value = stats.norm.sf(abs(z))*2

# построим доверительный интервал (ci) для разницы средних в абсолютном и относительном выражении

ci_lower = mean_difference - 1.96 * np.sqrt(variance) 
ci_upper = mean_difference + 1.96 * np.sqrt(variance)

ci_lower_relative = round(ci_lower * 100.0 / control_mean, 5)
ci_upper_relative = round(ci_upper * 100.0 / control_mean, 5)
```

В первом комментарии оставила ссылки на статьи, где более подробно разбирается вопрос (это на случай, если интересно погрузиться глубже).

P.S. в интернете есть готовые модули/пакеты, но мне приятнее видеть чистые формулы и понимать, как все работает, поэтому я использую собственную реализацию расчетов там, где это возможно и уместно.