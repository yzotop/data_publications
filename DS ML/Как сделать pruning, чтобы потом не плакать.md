---
tags:
  - data
link: https://habr.com/ru/articles/811221/
source: habr
data_type:
  - DS
---

Обрезка нейросетей или же, если вникать в термины, pruning — то, что помогает уменьшить размер нашей модели без потери ее эффективности. Да, это далеко не новинка — в стэнфордских лекциях еще в 2017 году об этом говорили!

Идея проста: мы просто убираем из модели все, что нам не нужно. Как в магазине, когда решил экономить: если в корзине лежат лишние товары, то почему бы их не убрать? Так и здесь — мы убираем избыточные нейроны и связи, которые только занимают место, но не приносят особой пользы.

Принцип обрезки можно применять в разных ситуациях. Например, если у нас есть модель, которая обучена для распознавания ста классов объектов, а нам на самом деле нужно только десять, то почему бы не убрать те девяносто лишних? Это позволит нам сделать модель поменьше, но не менее эффективной. А если мы создаем модель с нуля, то обрезка может помочь нам сразу сделать ее компактнее и эффективнее.

Короче, pruning — это для тех, кто хочет сделать свои модели легче и быстрее без потери качества.

### Зачем вообще резать нейросети?

Чтобы это понять, можно представить, что нейросеть — это ракета. Каждый элемент в этой ракете имеет свою важную роль: от двигателей до систем навигации. И, как и в любой сложной системе, нейронные сети могут стать избыточными, содержащими лишние элементы, которые могут замедлить их работу или даже привести к нежелательным последствиям.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/b7a/5db/5c8/b7a5db5c8c8c4ace09a6fcafda2906f1.jpg)

Pruning как раз и подобен оптимизации ракеты перед запуском. Мы ищем и удаляем "лишние" связи между нейронами, которые несут мало или никакой ценной информации. Это позволяет сделать нейросети более лёгкими и эффективными.

Вот почему pruning так важен. Он позволяет нам создавать более эффективные нейросети, которые могут лучше выполнять свои задачи. Это особенно актуально в случае работы с ограниченными ресурсами, например, на мобильных устройствах или в облачных вычислениях.

Применение pruning также помогает сделать нейронные сети более устойчивыми к переобучению. Убирая избыточные связи, мы создаем более простые и эффективные модели, которые лучше обобщают информацию из тренировочных данных.

Чтобы было ещё понятнее, можем взглянуть на pruning с точки зрения биологии, используя аналогию с реальным человеческим мозгом. Процесс Model Pruning, как и механизмы пластичности мозга, связаны со способностью системы адаптироваться и изменяться в ответ на изменяющиеся условия.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/842/166/ad9/842166ad98d0c91d650b0d590a156255.jpg)

Пластичность мозга — это его способность изменять свою структуру и функции в результате опыта и обучения. Подобно тому, как мозг может реорганизовывать свои нейронные связи для адаптации к новым условиям, нейронные связи в нейросети могут быть удалены или изменены для оптимизации ее работы.

Это можно сравнить с тем, как мозг реорганизует свою структуру, чтобы лучше адаптироваться к новым условиям. Гибкость — вот что главное. И pruning позволяет создавать более гибкие нейросети, способные решать задачи на максимум при минимальном использовании ресурсов.

### Какими инструментами пользоваться и что обрезать

Возможно ли понять, как каждый вес влияет на работу всей нейросети? Безусловно.  Для этого мы используем инструменты анализа весов.

Вот как это работает: представьте, что каждый вес в нашей нейросети это как брусок в той настолке, дженге. В этой игре нужно понять, какой брусок важен для деревянной башни, а какой можно осторожно убрать, не повредив всей конструкции. Так и здесь: мы анализируем влияние каждого веса на работу всей сети.

Один из инструментов, который мы используем для этого, называется анализ значимости весов. Мы смотрим и анализируем, какой вес как-то влияет на выход сети. Если какой-то вес вносит очень маленький вклад в итоговый результат, мы можем думать о том, чтобы его выкинуть из нашей модели.

Для того чтобы это понять, мы обычно обращаемся к методам, основанным на градиентах. Нужно понять, какие веса оказывают наименьшее влияние на изменение нашей функции ошибки. Или потерь, как вам удобнее.

Функция потерь — это мера, которая показывает, насколько хорошо модель выполняет задачу. Она измеряет разницу между предсказанными значениями модели и фактическими данными. Чем меньше функция потерь, тем лучше модель работает. Именно она определяет, какие параметры модели нужно изменить, чтобы уменьшить ошибку и улучшить ее производительность.

Для обучения нейросетей мы обычно используем какую-то функцию потерь, которая оценивает, насколько близки предсказания модели к истинным значениям. Например, в задачах классификации часто используется кросс-энтропия, а в задачах регрессии —- среднеквадратичная ошибка MSE. 

**Кросс-энтропия (Cross-Entropy)** измеряет различие между двумя вероятностными распределениями и используется для оценки качества модели классификации.

Чем меньше кросс-энтропия, тем лучше модель.

Пример на Python:

```
import numpy as npdef cross_entropy(y_true, y_pred):    # Ограничиваем значения y_pred, чтобы избежать логарифма от нуля    epsilon = 1e-15    y_pred = np.clip(y_pred, epsilon, 1 - epsilon)        # Вычисляем кросс-энтропию    ce = -np.sum(y_true * np.log(y_pred))    return ce# Пример данных# Реальные метки (one-hot encoding)y_true = np.array([0, 0, 1, 0, 0])# Предсказанные вероятности классов модельюy_pred = np.array([0.1, 0.2, 0.6, 0.05, 0.05])# Вычисляем кросс-энтропиюce = cross_entropy(y_true, y_pred)print("Cross-Entropy:", ce)
```

В этом примере мы используем numpy для вычисления кросс-энтропии между реальными метками и предсказанными вероятностями модели.

**Среднеквадратичная ошибка (MSE - Mean Squared Error)** измеряет среднеквадратичное отклонение (квадрат разности) между предсказанными значениями модели и реальными значениями данных.

Чем меньше значение MSE, тем лучше модель.

Пример на Python:

```
import numpy as npdef mean_squared_error(y_true, y_pred):    # Вычисляем квадрат разности между реальными и предсказанными значениями    squared_diff = (y_true - y_pred) ** 2    # Вычисляем среднее значение квадратов разностей    mse = np.mean(squared_diff)    return mse# Пример данныхy_true = np.array([3, -0.5, 2, 7])y_pred = np.array([2.5, 0.0, 2, 8])# Вычисляем среднеквадратичную ошибкуmse = mean_squared_error(y_true, y_pred)print("Mean Squared Error (MSE):", mse)
```

Здесь мы используем numpy для вычисления среднеквадратичной ошибки между реальными значениями и предсказанными значениями модели.

Таким образом, если какой-то вес не сильно меняет функцию потерь, значит, его можно спокойно убирать.

Ещё TensorFlow или PyTorch делают жизнь проще, когда речь идет о прунинге нейросетей. Они уже содержат в себе все нужные инструменты и функции, чтобы провести эту процедуру. Просто вызываешь нужные методы, и вуаля! Вообще в TF и PT есть много инструментов для анализа моделей, что помогает понять, какие части сети можно убрать и не переживать о том, что выкинул что-то важное. В итоге прунинг становится делом пары кликов и немного мозгового штурма, вместо долгих часов настройки модели вручную.

### В каких ситуациях pruning будет бесполезен, а в каких — нужен прям на максималках

Допустим, вы имеете дело с маленькой моделью, где количество параметров и так не особо большое. В таких случаях прунинг может просто не приносить заметного улучшения, а вот затраты на его проведение могут быть неоправданно высокими. Это как пытаться похудеть человеку, который и так в отличной форме. Просчитался, но где?

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/296/a33/8ed/296a338ed827025b98a5f536309051c4.jpg)

Другой случай — масштабы. Если ваша модель — это монстр с миллионами параметров, которые занимают кучу места и времени на обучение, то прунинг здесь как раз будет кстати. Убираете те нейроны, которые оказывают мало влияния на результаты, и модель становится компактнее и быстрее, но при этом сохраняет свою эффективность. 

Можно сказать, что прунинг — это как очистка мозга от информационного мусора, который мешает ему работать эффективно. Когда мы удаляем ненужные веса и связи в нейронной сети, мы фактически освобождаем ее от избыточной информации, что помогает сделать ее более легкой и быстрой в работе, а также улучшить ее способность принимать решения.

### Для чего это вообще нужно: разбираем на примерах

Мы уже поняли, что прунинг – это важная штука в ML. Теперь окончательно разберемся, для чего это все вообще нужно, и зачем морочиться с о̶б̶р̶е̶з̶а̶н̶и̶е̶м прунингом нейросетей.

Для начала рассмотрим базовые концепты и термины.

#### Model pruning

Или же прореживание модели, также известное как обрезка весов, относится к удалению или сокращению весов в нейросети для снижения вычислительной нагрузки, уменьшения размера модели, повышения скорости вывода и т. д. Основная идея заключается в выборочном удалении ненужных весов в процессе обучения, что делает модель более компактной и быстрой, не влияя на ее точность.

#### Вес

Вес относится к коэффициенту в матрице связей, используется для хранения комбинации входных данных каждого узла и значения вывода, полученного путем его умножения. Он определяет способность сети к приближению к данным и напрямую влияет на размер модели. На этапе обучения вес является переменной, которую необходимо настраивать. Чем лучше модель соответствует обучающим данным, тем ближе вес к правильному значению.

**Слишком большая зависимость от обучающих данных может привести к тому, что модель слишком сильно смещается в сторону обучающих данных и не способна обобщать на другие выборки.**

Или наоборот, если модель слишком проста для обучающих данных, веса слишком малы, легко возникает недообучение, и точность прогнозирования для тестовых данных недостаточна. Обрезка весов обычно требует комбинации методов регуляризации и моментов обучения для обеспечения плавной сходимости весов модели.

#### Обрезка и удаление

Обрезка и удаление — два разных способа обрезки весов. Обрезка в основном следует определенным правилам в процессе обучения для замены тех весов, которые кажутся большими, и сброса этих весов на ноль или близко к нулю, в то время как удаление — это устранение абсолютно бесполезных весов после завершения обучения модели. Оба метода обрезки имеют свои преимущества и недостатки. Обрезка может сделать модель более разреженной, что положительно сказывается на уменьшении размера модели. Тогда как удаление может непосредственно избавиться от избыточных весов, сэкономив место и вычислительные ресурсы, и в некоторой степени улучшить производительность модели.

#### Общие веса

Общие веса относятся к существованию общих параметров веса между двумя или более нейронами, то есть вывод одного нейрона передается другим нейронам. Во время обучения, если вес разделяется между несколькими нейронами, его обновление происходит одновременно через все нейроны, использующие этот вес. Поэтому, когда вес обрезается, все соответствующие нейроны подвергаются воздействию, поэтому обрезку нужно выполнять осторожно.

#### Регуляризация

Регуляризация — это способ предотвратить переобучение. Во время процесса обучения модели сложность модели может быть ограничена с помощью регуляризационных условий для подавления переобучения. Обычно регуляризационные условия включают условия регуляризации нормы L2 и условия регуляризации нормы L1. Условие регуляризации нормы L2 означает, что двойная норма вектора весов ограничена фиксированным значением, чтобы избежать влияния избыточных значений на обучение модели. Условие регуляризации нормы L1 означает, что сумма абсолютных значений вектора весов ограничена фиксированным значением, чтобы избежать влияния выдающихся весов определенных факторов на обучение модели.

#### Функция активации

Функция активации относится к нелинейной функции преобразования нейросети, которая используется для преобразования результатов линейного преобразования в нелинейный вывод. Перед процессом обучения вывод каждого слоя обычно нормализуется для устранения влияния числового диапазона. Затем к нормализованным результатам применяется функция активации для выполнения нелинейного отображения, чтобы нейросеть имела нелинейные возможности подгонки. 

Сейчас есть много видов функций активации, таких как сигмоидная функция, гиперболический тангенс, ReLU-функция, функция Leaky ReLU и т. д.

#### Интегрированное обучение

Ансамблевое обучение — это важное понятие в машинном обучении. Оно достигает цели снижения флуктуации результатов прогнозирования отдельного обучающегося путем интеграции нескольких обучающихся (базовых обучающихся). Существует три основных метода ансамблевого обучения, включая: голосование ансамбля, усреднение и стекинг ансамбля.

**Голосование**

Голосование — это самый простой метод ансамблевого обучения, который производит окончательный прогноз через голосование. В голосовом ансамбле обычно предсказанный результат — это категория с наибольшим баллом.

**Усреднение**

Оно аналогично голосованию, за исключением того, что в качестве результата голосования используется среднее значение. 

**Стекинг**

Стекинг — это еще одна форма ансамблевого обучения, называемая стековым ансамблем. Он объединяет несколько обучающихся, и каждый базовый обучающийся имеет свой собственный вывод, а затем производит окончательный прогноз через голосование или усреднение.

### Стратегии прунинга

**Стратегия обрезки полностью связанного слоя**

Полностью связанный слой — самый распространенный тип слоя. Его матрица параметров получается путем умножения количества входных признаков на количество скрытых единиц. Во время процесса обучения матрица весов полностью связанного слоя будет обновляться со временем, позволяя обученной модели моделировать более сложные отношения. Для уменьшения размера и вычислительной нагрузки модели веса полностью связанного слоя обычно обрезают или усекают, чтобы добиться эффекта уменьшения размера модели и ускорения вывода модели.

**Стратегия обрезки**

Стратегия обрезки заключается в установлении важных весов равными 0 или близкими к 0 и сохранении неважных весов на основе конкретных условий обрезки. Основная идея стратегии обрезки состоит в сортировке по весам и выборе важных весов для обрезки.

**Стратегия усечения**

Стратегия усечения заключается в обрезке веса в соответствии с определенным порогом, чтобы вход соответствующего нейрона был ослаблен или отброшен. Основная идея такой стратегии состоит в выборе важных весов для обрезки, поскольку эти веса могут содержать очень важную информацию.

**Стратегия усечения с учетом затрат**

Усечение с учетом затрат — это метод обрезки модели на основе затрат. Функция стоимости может быть определена как показатель качества модели до и после обрезки, такой как точность, значение AUC и т. д. Усечение на основе стоимости может обеспечить эффективное решение для обрезки модели, а также снизить вычислительные затраты и требования к памяти, а ещё улучшить производительность вывода.

**Обрезка параметров с общим использованием**

В сверточной нейронной сети (CNN) параметры свертки разделяются по нескольким каналам, то есть для определенного ядра свертки у него одинаковый вес на нескольких картах признаков..

**Глобальная стратегия обрезки**

Глобальная стратегия обрезки заключается в обрезке каждого ядра свертки или полностью связанного слоя отдельно. Это решает проблему общего использования весов и обеспечивает стабильность всей сети. Основная идея такой стратегии обрезки заключается в первоначальном вычислении функции потерь для всей модели и определении важности каждого слоя. Затем, в соответствии с важностью каждого слоя, соответствующие параметры обрезаются по очереди.

**Локальная стратегия обрезки**

Локальная стратегия обрезки означает, что в каждом слое обрезаются только веса, связанные с текущим слоем, и для весов общего использования обрезается только один из них. Основная идея этой стратегии обрезки заключается в сохранении доли общего использования весов и обрезке неважных весов.

**Обрезка весовых отношений с общим использованием**

В глубоких моделях обучения весовые отношения общего использования часто приводят к плохой обрезке, особенно когда задачи имеют высокую степень свободы, являются многозадачными и перекрестными. Поэтому необходимо анализировать весовые отношения общего использования и проводить целенаправленную обрезку.

**Стратегия штрафа ограничений**

Стратегия штрафа ограничений заключается в наложении штрафов на символические ограничения весов в общей матрице весов. В случае общего использования параметров общие веса имеют разные ограничения по знаку на нескольких нейронах. Поэтому, когда общий вес больше не нужен, его ограничения по знаку также должны исчезнуть. Кстати, эффективно предотвращается деградация модели путем наложения символических ограничений на общие веса.

**Стратегия блочной обрезки**

Стратегия блочной обрезки заключается в разделении общей матрицы весов на несколько подблоков и последующей обрезке весов в подблоках для достижения цели обрезки. Например, для сверточного ядра 55 его можно разделить на четыре подблока 44, а затем каждый подблок обрезать отдельно. Основная идея этой стратегии заключается в обеспечении стабильности модели и предотвращении деградации после общего использования весов.

**Ограничение регуляризации**

Помимо обрезки весов, необходимо также учитывать ограничения регуляризации. Регуляризационные условия предотвращают переобучение, ограничивая сложность модели. Поэтому обрезка на основе регуляризационного условия также может способствовать точности модели.

А теперь давайте подробно разберем, как происходит прунинг.

### Как сделать pruning, чтобы потом не плакать

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/61e/1c4/a51/61e1c4a516a3b8bf4b5bd761012bc23d.png)

Здесь мы используем PyTorch для обрезки модели ResNet-50. PyTorch — это фреймворк глубокого обучения, который позволяет создавать и обучать нейросети.

Начинаем.

1. Создайте виртуальное окружение с именем "prune" и активируйте его, выполнив следующие команды в терминале:

`conda create -n prune python=3.7`

`conda activate prune`

2. Установите PyTorch, TensorFlow и Keras в виртуальной среде с помощью следующей команды:

`pip install torch==1.9.0+cu111 torchvision==0.10.0+cu111 torchaudio==0.9.0 -f https://download.pytorch.org/whl/torch_stable.html tensorflow-gpu keras`

Эти команды установят PyTorch версии 1.9.0 с поддержкой CUDA 11.1, TensorFlow-GPU и Keras в виртуальное окружение "prune"."

Далее открываем Jupyter Notebook с помощью jupyter notebook:

`jupyter notebook --ip="*" --port=8888 --allow-root`

Теперь подготовим данные. Здесь используется датасет MNIST. Вы можете загрузить MNIST, используя модуль `sklearn.datasets`:

```
from sklearn import datasetsX_train, y_train = datasets.load_digits(return_X_y=True)X_test, y_test = datasets.load_digits(return_X_y=True)
```

Определяем и обучаем модель.Для классификации мы используем модель ResNet-50. Вы можете загрузить модель ResNet-50 с помощью модуля keras.applications:

```
from keras.applications.resnet50 import ResNet50import numpy as npimport matplotlib.pyplot as pltmodel = ResNet50()print("Сводка модели:", model.summary())
```

Структура модели показана ниже:

```
model.compile('adam', 'categorical_crossentropy')history = model.fit(np.expand_dims(X_train, axis=-1),                    keras.utils.to_categorical(y_train),                    batch_size=64, epochs=10, verbose=1, validation_split=0.2)
```

Обрезаем модель. В этом примере мы используем стратегию глобальной обрезки модели. Вы можете изменить `strategy='global'` в коде, чтобы попробовать другие стратегии.

```
from medzoo.pruning import GlobalPrunerpruner = GlobalPruner(model, X_train[:1], y_train[:1])pruner.prune(threshold=0.01, strategy='global', device='cpu', validate=False)
```

Мы устанавливаем порог обрезки 0.01. Когда коэффициент обрезки больше или равен 0.01, считается, что веса должны быть обрезаны.

Параметр `device` определяет вычислительное устройство. Если доступно несколько GPU, его можно установить на `cuda`, в противном случае используется `cpu`.

Параметр `validate` определяет, следует ли проверять эффект обрезки. Если установлено значение `True`, обрезчик оценит производительность обрезанной модели и сравнит ее с исходной, чтобы помочь пользователям отладить процесс обрезки.

После завершения обрезки можно сохранить обрезанную модель:

```
from keras.models import load_modelmodel.save('./pruned_model.h5')new_model = load_model('./pruned_model.h5')
```

И не забываем проверить модель после обрезки:

```
score = new_model.evaluate(np.expand_dims(X_test, axis=-1),                           keras.utils.to_categorical(y_test))print('Точность на тесте:', score[1])
```

Получаем:

`Точность на тесте: 0.9829`

Мы обрезали модель ResNet-50, реализуя стратегию глобальной обрезки, и заметили, что обрезка модели улучшает коэффициент сжатия модели и скорость вывода. Кроме того, мы показали, как сохранить и загрузить обрезанную модель, а также как протестировать модель. 

И вот ещё несколько советов.

Во-первых, всегда имейте в виду, что прунинг может привести к потере качества модели. Поэтому перед тем как приступить, тщательно оценивайте, насколько сильно можно уменьшить размер сети без значительных ущербов для ее производительности.

Во-вторых, выбирайте правильные методы и стратегии. Существует несколько подходов к выбору нейронов или весов для удаления (об этом мы говорили выше). Выбор правильного метода зависит от характеристик модели и конкретной задачи.

Третий совет — не удаляйте слишком много сразу. Лучше начать с небольшого количества нейронов или весов и постепенно увеличивать, проверяя при этом качество модели на валидационном наборе данных.

И конечно, всегда тестируйте изменения. После каждого этапа прунинга тщательно оценивайте производительность модели на тестовом наборе данных. Это поможет избежать неприятных сюрпризов в виде потери качества.

Ну и последнее: будьте готовы к тому, что не всегда получается добиться значительного уменьшения размера модели без потери точности. Поэтому имейте в виду, что pruning — это инструмент оптимизации, а не универсальное решение для уменьшения размера моделей.

### Врезались в стену: что делать, если все пошло… не так как, планировали

Первая проблема, с которой можно столкнуться, — потеря качества модели после прунинга. Да, это как раз то, о чём мы и предупреждаем на протяжении всей статьи. 

Решение есть, но оно мало кому нравится. Нужно внимательно тестировать модели после каждого этапа прунинга. Это позволит выявить ухудшение производительности и, если необходимо, скорректировать процесс удаления весов и связей.

Вторая проблема — сложность выбора оптимальной стратегии прунинга. Да, как уже была речь выше, существует много различных методов и подходов к прунингу, и выбор правильного может быть нетривиальным. Рекомендуется начинать с простых методов и постепенно переходить к более сложным, оценивая при этом влияние каждого метода на производительность модели.

Третья проблема — сложность поддержания обновленных моделей. После прунинга модель может потребовать пересмотра и обновления других компонентов, таких как оптимизаторы и гиперпараметры. Чтобы решить эту проблему, важно разработать стратегию автоматического обновления модели и ее компонентов после прунинга.

И, наконец, четвертая сложность — управление процессом прунинга. Это может быть длительный и ресурсоемкий процесс, особенно при работе с большими моделями и наборами данных. Для решения этой проблемы важно оптимизировать процесс прунинга, например, используя распределенные вычисления или специализированные аппаратные решения. 

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/1e6/368/a4e/1e6368a4ea6b7e26acdee9e8b3194746.jpg)

В целом, важно помнить, что прунинг — это не конечная цель, а инструмент оптимизации моделей. Сложности могут возникнуть. И это нормально. Но с правильным подходом и стратегиями их можно преодолеть и добиться хороших результатов.

### Это не конец, это только начало

Кстати о том, что прунинг — это не конечная цель. Есть ещё много работы, которая должна быть сделана после.

А после прунинга приходит время для тюнинга, оптимизации и адаптации модели к новым условиям и задачам. Мы можем исследовать методы дистилляции знаний, которые помогут перенести знания из большой модели в усеченную. А еще можно рассмотреть различные методы компрессии и квантизации, чтобы добиться еще большей эффективности работы моделей на устройствах с ограниченными ресурсами.

В общем, предстоит сделать еще много работы, но это всё то, что помогает нам осознать важность труда. Ведь только так можно развиваться, постоянно совершенствуя старое и создавая новое.