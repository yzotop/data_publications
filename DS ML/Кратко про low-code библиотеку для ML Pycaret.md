---
tags:
  - data
data_type:
  - DS
company: Otus
source: habr
link: https://habr.com/ru/companies/otus/articles/839582/
author: Otus
---

Привет, Хабр!

PyCaret — это open-source библиотека, которая предлагает low-code подход к созданию, обучению и внедрению моделей ML. Она позволяет провести весь процесс — от подготовки данных до развертывания модели в продакшн — всего за несколько строк кода.

## Основные возможности

PyCaret разделен на несколько модулей, каждый из которых предназначен для выполнения определенного типа задач:

1. `pycaret.classification`: для задач, где нужно предсказать категорию или класс.
    
2. `pycaret.regression`: для задач предсказания числового значения.
    
3. `pycaret.clustering`: для сегментации данных на группы с схожими характеристиками.
    
4. `pycaret.anomaly`: для выявления нетипичных данных, которые значительно отличаются от остальных.
    
5. `pycaret.time_series`: для задач прогнозирования значений во времени.
    
6. `pycaret.nlp`: для задач обработки естественного языка.
    

Первый шаг при работе с PyCaret — это инициализация окружения с помощью функции `setup()`. Эта функция выполняет предобработку данных.

```
from pycaret.classification import setup# загрузка данныхdata = pd.read_csv('data.csv')# инициализация экспериментаexp = setup(data=data, target='target_column',             normalize=True,             train_size=0.8,             session_id=123)
```

Код выполнит след. шаги:

- Загружает данные и задает целевую переменную.
    
- Автоматически разбивает данные на обучающую (80%) и тестовую (20%) выборки.
    
- Применяет нормализацию числовых признаков.
    
- Фиксирует случайное значение для воспроизводимости результатов с помощью `session_id`.
    

После выполнения `setup()`, PyCaret возвращает множество полезных объектов, которые можно использовать в дальнейшем. Например, можно получить доступ к данным после предобработки:

```
X_train = get_config('X_train')X_test = get_config('X_test')
```

Функция `compare_models()` — это, на мой взгляд, одна из самых мощных функций PyCaret. Она позволяет автоматически обучить и оценить несколько моделей, используя кросс-валидацию, и возвращает лучшую модель на основе выбранной метрики.

```
from pycaret.classification import compare_models# сравнение всех доступных моделей и выбор лучшейbest_model = compare_models()
```

По умолчанию, `compare_models()` выполняет оценку всех доступных моделей и сортирует их по метрике качества. Можно настроить поведение этой функции, например, указав количество моделей, которые нужно вернуть:

```
# возвращаем топ-3 моделиtop3_models = compare_models(n_select=3)
```

После выбора лучшей модели можно провести доп. тюнинг её гиперпараметров с помощью функции `tune_model()`. Эта функция автоматизирует процесс поиска оптимальных параметров.

```
from pycaret.classification import tune_model# тюнинг лучшей моделиtuned_model = tune_model(best_model)
```

После настройки и тюнинга модели, можно ее визуализировать и оценить её производительность с помощью функции `evaluate_model()`.

```
from pycaret.classification import evaluate_model# оценка моделиevaluate_model(tuned_model)
```

Функция `blend_models()` используется для создания ансамблей моделей путем их объединения. Она позволяет выбрать несколько моделей и комбинировать их прогнозы с использованием различных стратегий, таких как усреднение или взвешенное усреднение:

```
from pycaret.classification import setup, compare_models, blend_models# Шаг 1: Инициализация экспериментаexp = setup(data=data, target='target_column', normalize=True, session_id=123)# Шаг 2: Сравнение моделей и выбор топ-3top3_models = compare_models(n_select=3)# Шаг 3: Ансамблирование топ-3 моделейblended_model = blend_models(estimator_list=top3_models)
```

Функция `stack_models()` используется для создания многоуровневых ансамблей, где выходы базовых моделей передаются на вход мета-модели. Этот подход позволяет учесть ошибки базовых моделей и улучшить финальный прогноз:

```
from pycaret.classification import setup, compare_models, stack_models# Шаг 1: Инициализация экспериментаexp = setup(data=data, target='target_column', normalize=True, session_id=123)# Шаг 2: Сравнение моделей и выбор топ-5top5_models = compare_models(n_select=5)# Шаг 3: Стекинг моделейstacked_model = stack_models(estimator_list=top5_models, meta_model=None)
```

**Пример задачи регрессии** на наборе данных. Весь процесс от предобработки данных до выбора и тюнинга модели занимает всего несколько строк кода:

```
import pandas as pdfrom pycaret.regression import setup, compare_models, tune_model, save_model# Шаг 1: Загрузка данныхdata = pd.read_csv('diamonds.csv')# Шаг 2: Инициализация экспериментаexp = setup(data=data, target='price', normalize=True, silent=True)# Шаг 3: Сравнение моделейbest_model = compare_models()# Шаг 4: Тюнинг лучшей моделиtuned_model = tune_model(best_model)# Шаг 5: Сохранение моделиsave_model(tuned_model, model_name='diamond_price_model')
```

Для решения других задач (например, классификации, кластеризации, или работы с временными рядами) нужно просто заменить модуль импорта `pycaret.regression` на соответствующий модуль, например, `pycaret.classification` для классификации, и так далее. Логика работы остается такой же.

## Пример применения

Разберем кейс, связанный с прогнозированием цены на жилье.

Для начала загрузим и подготовим набор данных о ценах на жилье. Будем использовать дефолт датасет `house-prices`:

```
import pandas as pd# Загрузка данныхdata = pd.read_csv('house-prices.csv')
```

Инициализируем окружение:

```
from pycaret.regression import setup# инициализацияexp = setup(data=data, target='SalePrice',             normalize=True,             polynomial_features=True,             feature_interaction=True,             log_experiment=True,             experiment_name='house_price_prediction',            silent=True)
```

Сравним модели с `compare_models()`

На этом этапе автоматически обучим и сравним несколько моделей ML, чтобы выбрать наилучшую:

```
from pycaret.regression import compare_models# сравнение всех доступных моделей и выбор лучшейbest_model = compare_models()
```

Чтобы улучшить производительность выбранной модели, проведем тюнинг её гиперпараметров.

```
from pycaret.regression import tune_model# тюнинг лучшей моделиtuned_model = tune_model(best_model)
```

После тюнингаможно визуализировать и оценить работу модели с помощью различных графиков:

```
from pycaret.regression import plot_model, evaluate_model# Построение графиков для оценки моделиplot_model(tuned_model, plot='residuals')plot_model(tuned_model, plot='error')evaluate_model(tuned_model)
```

Теперь теперь можно использовать модель для предсказания на новых данных:

```
from pycaret.regression import predict_model# прогнозирование на новых данныхnew_data = pd.read_csv('new_house_data.csv')predictions = predict_model(tuned_model, data=new_data)print(predictions)
```

Сохраним модель для дальнейшего использования или развертывания в продакшн:

```
from pycaret.regression import save_model# Сохранение моделиsave_model(tuned_model, model_name='final_house_price_model')
```

---

Подробнее с библиотекой можно [ознакомиться здесь](https://pycaret.gitbook.io/docs).

А другие библиотеки и практические инструменты коллеги из OTUS рассматривают в рамках практических онлайн-курсов. [Подробнее в каталоге](https://otus.pw/DGa9/).