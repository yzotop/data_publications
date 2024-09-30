---
tags:
  - data
link: https://habr.com/ru/companies/ods/articles/828458/
source: habr
data_type:
  - MLOps
rep_link: https://github.com/egorborisov/mlflow_example
company: ODS
author: ODS
---
[GitHub - egorborisov/mlflow\_example: Machine learning lifecycle focusing on integration with MLflow.](https://github.com/egorborisov/mlflow_example)

MLflow - это инструмент для управления жизненным циклом машинного обучения: отслеживание экспериментов, управление и деплой моделей и проектов. В этом руководстве мы посмотрим, как организовать эксперименты и запуски, оптимизировать гиперпараметры с помощью optuna, сравнивать модели и выбирать лучшие параметры. Также рассмотрим логирование моделей, использование их в разных форматах, упаковку проекта в MLproject и установку удаленного Tracking Server MLflow.

> Вы можете просто прочитать статью или повторить все шаги локально. Также по ссылке [репозиторий](https://github.com/egorborisov/mlflow_example), который является расширенной версией этого гайда на английском в README.


```
Подготовка env

Сначала нам нужно установить `conda` и создать новоет окружение:


name: mlflow-examplechannels:  - conda-forge  - defaultsdependencies:  - python=3.11.4  - pip=24.0  - mlflow=2.14.2  - xgboost=2.0.3  - jupyter=1.0.0  - loguru=0.7.2  - shap=0.45.1  - pandas=2.2.2  - scikit-learn=1.5.1  - scipy=1.14.0  - numpy=1.26.4  - jupytext=1.16.3  - psutil=6.0.0  - boto3=1.34.148  - psycopg2-binary=2.9.9  - pip:    - optuna==3.6.1    - mlserver==1.3.5    - mlserver-mlflow==1.3.5    - mlserver-xgboost==1.3.5


Можно установить вручную или сохранить этот файл локально и вызвать:


conda env create -f conda.yaml

Теперь можно использовать это окружение в IDE или в терминале:

conda activate mlflow-example
```


## MLflow UI

Выполните в терминале `mlflow ui`, чтобы запустить MLflow на localhost:5000.

Mlflow будет запущен в локальном режиме, по умолчанию он создаст папку `mlruns` для хранения артефактов и метаинформации.

![MLflow UI](https://habrastorage.org/r/w1560/getpro/habr/upload_files/a09/fc3/ec9/a09fc3ec9340d0fc0839a1e78467917f.png "MLflow UI")

MLflow UI

### Mlflow Projects

MLflow Project - это подход к структурированию проекта, следуя конвенциям, которые упрощают запуск проекта для других разработчиков (или автоматических инструментов). Каждый проект — это директория файлов или git-репозиторий, содержащий ваш код. MLflow может запускать эти проекты, основываясь на определенных правилах организации файлов в директории.

Файл MLproject помогает MLflow и другим пользователям понять и запустить ваш проект, указывая окружение, точки входа и возможные параметры для настройки:

```
name: Cancer_Modelingconda_env: conda.yamlentry_points:  data-preprocessing:    parameters:      test-size: {type: float, default: 0.33}    command: "python data_preprocessing.py --test-size {test-size}"  hyperparameters-tuning:    parameters:      n-trials: {type: int, default: 10}    command: "python hyperparameters_tuning.py --n-trials {n-trials}"  model-training:    command: "python model_training.py"  data-evaluation:    parameters:      eval-dataset: {type: str}    command: "python data_evaluation.py --eval-dataset {eval-dataset}"
```

Мы можем запускать эндпоинты проекта, используя интерфейс командной строки (CLI), либо API на Python.

### Mlflow Experiments and Mlflow Runs

MLflow Experiments и MLflow Runs - это основные абстракции для структурирования проекта. Давайте рассмотрим пример разработки модели на открытом датасете с данными по забалеванию раком.

#### Data preprocessing

Этот шаг обычно более сложный, но здесь мы просто загрузим набор данных, разделим его на обучающую и тестовую выборки, залогируем несколько метрик в MLflow (количество образцов и признаков) и передадим сами наборы данных в артефакты MLflow.

```
import sysimport argparseimport mlflowimport warningsimport pandas as pdfrom sklearn.model_selection import train_test_splitfrom sklearn import datasetsfrom loguru import loggerfrom config import config# set up logginglogger.remove()logger.add(sys.stdout, format="{time:YYYY-MM-DD HH:mm:ss} | {level} | {message}")warnings.filterwarnings('ignore')def get_cancer_df():    cancer = datasets.load_breast_cancer()    X = pd.DataFrame(cancer.data, columns=cancer.feature_names)    y = pd.Series(cancer.target)    logger.info(f'Cancer data downloaded')    return X, yif __name__ == '__main__':        parser = argparse.ArgumentParser()    parser.add_argument("--test-size", default=config.default_test_size, type=float)    TEST_SIZE = parser.parse_args().test_size            logger.info(f'Data preprocessing started with test size: {TEST_SIZE}')            # download cancer dataset    X, y = get_cancer_df()    # add additional features    X['additional_feature'] = X['mean symmetry'] / X['mean texture']    logger.info('Additional features added')    # log dataset size and features count    mlflow.log_metric('full_data_size', X.shape[0])    mlflow.log_metric('features_count', X.shape[1])    # split dataset to train and test part and log sizes to mlflow    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=TEST_SIZE)    mlflow.log_metric('train_size', X_train.shape[0])    mlflow.log_metric('test_size', X_test.shape[0])        # log and register datasets    train = X_train.assign(target=y_train)    mlflow.log_text(train.to_csv(index=False),'datasets/train.csv')    dataset_source_link = mlflow.get_artifact_uri('datasets/train.csv')    dataset = mlflow.data.from_pandas(train, name='train', targets="target", source=dataset_source_link)    mlflow.log_input(dataset)    test = X_test.assign(target=y_test)    mlflow.log_text(test.to_csv(index=False),'datasets/test.csv')    dataset_source_link = mlflow.get_artifact_uri('datasets/test.csv')    dataset = mlflow.data.from_pandas(train, name='test', targets="target", source=dataset_source_link)    mlflow.log_input(dataset)        logger.info('Data preprocessing finished')
```

Чтобы выполнить этот код сохраните его в файл `data-preprocessing.py` рядом с файлом `MLproject` и в командной строке вызовете:

```
mlflow run . --entry-point data-preprocessing --env-manager local --experiment-name Cancer_Classification --run-name Data_Preprocessing -P test-size=0.33
```

Теперь этот запуск должен быть доступен через ui. Секция datasets used заполнена благодаря mlflow.data api. Однако важно понимать, что это метаданные о даных, а не сами данные.

> Модуль mlflow.data отслеживает информацию о наборах данных во время обучения и оценки модели: признаки, целевые значения, предсказания, название, схема и источник. Эти метаданные логируются с помощью `mlflow.log_input()`.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/671/6ff/5ba/6716ff5ba9065de42d874f252498b5dc.png)

#### Hyperparameters tuning

В этой части мы используем `optuna` для нахождения лучших гиперпараметров для XGBoost, используя встроенную кросс-валидацию для обучения и оценки модели. Кроме того, мы посмотрим, как собирать метрики во время процесса обучения.

Здесь мы используем `conda` в качестве env менджера: `mlflow` автоматически создаст для нас новый env из файла, указанного в конфигурации `MLproject`.

```
mlflow run . --entry-point hyperparameters-tuning --env-manager conda --experiment-name Cancer_Classification --run-name Hyperparameters_Search -P n-trials=10
```

Посмотрим на результаты в MLflow UI. Для структурирования проекта можно использовать вложенные запуски. Есть один основной запуск для настройки гиперпараметров, а все испытания собираются как вложенные запуски. MLflow также предоставляет возможность настраивать столбцы и порядок строк в этом представлении:

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/b45/008/221/b450082218632764ec0519e1145e558c.png)

В сhart view можно сравнить запуски и настроить различные диаграммы. Использование XGBoost callbacks для логирования метрик в процессе обучения модели позволяет создавать графики с количеством деревьев на оси x.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/cee/5da/212/cee5da21249cdd44051f57035acf177b.png)

Выберите несколько запусков, нажмите кнопку сравнения и выберите наиболее полезное представление. Функция mlflow compare может быть полезной при оптимизации гиперпараметров, так как помогает уточнить и корректировать границы возможных интервалов на основе результатов сравнения.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/6cd/f02/61b/6cdf0261b80c2cea0244a47a0dbc0d50.png)

Системные метрики также могут отслеживаться на протяжении всего запуска. Хотя это и не дает точной оценки реальных требований, в некоторых случаях все же может быть полезным.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/c5e/e88/bc9/c5ee88bc953fe7e595923f46d9fa9942.png)

#### Log and register model

Возможно, но не обязательно, сохранять модель для каждого эксперимента и запуска. В большинстве случаев лучше сохранять параметры, а затем, выбрав лучшие, выполнить дополнительный запуск для сохранения модели. Здесь мы следуем той же логике: используем параметры из лучшего запуска для сохранения финальной модели и регистриреум ее для версионирования и использования через короткую ссылку.

```
import osimport sysimport tempfileimport mlflowimport warningsimport loggingimport xgboost as xgbimport pandas as pdfrom loguru import logger# set up loggingwarnings.filterwarnings('ignore')logging.getLogger('mlflow').setLevel(logging.ERROR)logger.remove()logger.add(sys.stdout, format="{time:YYYY-MM-DD HH:mm:ss} | {level} | {message}")if __name__ == '__main__':    logger.info('Model training started')     mlflow.xgboost.autolog()    with mlflow.start_run() as run:        experiment_id = run.info.experiment_id                run_id = run.info.run_id        logger.info(f'Start mlflow run: {run_id}')                # get last finished run for data preprocessing        last_data_run_id = mlflow.search_runs(            experiment_ids=[experiment_id],            filter_string=f"tags.mlflow.runName = 'Data_Preprocessing' and status = 'FINISHED'",            order_by=["start_time DESC"]        ).loc[0, 'run_id']            # download train and test data from last run        with tempfile.TemporaryDirectory() as tmpdir:            mlflow.artifacts.download_artifacts(run_id=last_data_run_id, artifact_path='datasets/train.csv', dst_path=tmpdir)            mlflow.artifacts.download_artifacts(run_id=last_data_run_id, artifact_path='datasets/test.csv', dst_path=tmpdir)            train = pd.read_csv(os.path.join(tmpdir, 'datasets/train.csv'))            test = pd.read_csv(os.path.join(tmpdir, 'datasets/test.csv'))        # convert to DMatrix format        features = [i for i in train.columns if i != 'target']        dtrain = xgb.DMatrix(data=train.loc[:, features], label=train['target'])        dtest = xgb.DMatrix(data=test.loc[:, features], label=test['target'])        # get last finished run for hyperparameters tuning        last_tuning_run = mlflow.search_runs(            experiment_ids=[experiment_id],            filter_string=f"tags.mlflow.runName = 'Hyperparameters_Search' and status = 'FINISHED'",            order_by=["start_time DESC"]        ).loc[0, :]                # get best params        params = {col.split('.')[1]: last_tuning_run[col] for col in last_tuning_run.index if 'params' in col}        params.update(eval_metric=['auc', 'error'])        mlflow.log_params(params)                model = xgb.train(            dtrain=dtrain,            num_boost_round=int(params["num_boost_round"]),            params=params,            evals=[(dtest, 'test')],            verbose_eval=False,            early_stopping_rounds=10        )        mlflow.log_metric("accuracy", 1 - model.best_score)                # Log model as Booster        input_example = test.loc[0:10, features]        predictions_example = pd.DataFrame(model.predict(xgb.DMatrix(input_example)), columns=['predictions'])        mlflow.xgboost.log_model(model, "booster", input_example=input_example)        mlflow.log_text(predictions_example.to_json(orient='split', index=False), 'booster/predictions_example.json')        # Register model        model_uri = f"runs:/{run.info.run_id}/booster"        mlflow.register_model(model_uri, 'CancerModelBooster')                # Log model as sklearn completable XGBClassifier        params.update(num_boost_round=model.best_iteration)        model = xgb.XGBClassifier(**params)        model.fit(train.loc[:, features], train['target'])        mlflow.xgboost.log_model(model, "model", input_example=input_example)        # log datasets        mlflow.log_text(train.to_csv(index=False), 'datasets/train.csv')        mlflow.log_text(test.to_csv(index=False),'datasets/test.csv')        logger.info('Model training finished')        # Register the model        model_uri = f"runs:/{run.info.run_id}/model"        mlflow.register_model(model_uri, 'CancerModel')                logger.info('Model registered')
```

```
mlflow run . --entry-point model-training --env-manager conda --experiment-name Cancer_Classification --run-name Model_Training -P n-trials=10 
```

Благодаря функции `mlflow.xgboost.autolog()`, все метрики автоматически логируются, в том числе в процессе обучения:

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/0c6/8d0/c6b/0c68d0c6bdde3e1de348ac8766c7bc79.png)

После сохранения модели, мы можем получить доступ к странице артефактов внутри запуска. В артефактах можно хранить любые типы файлов, такие как пользовательские графики, текстовые файлы, изображения, наборы данных, скрипты Python и другие. Например, я конвертировал рабочий ноутбук в HTML и сохранил его как артефакт:

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/fe2/426/0b2/fe24260b2ad0a2aa72691688612f66c6.png)

Для каждой модели MLflow автоматически создаёт YAML конфигурационный файл, называемый MLmodel. Этот файл можно просмотреть в интерфейсе MLflow или скачать и изучить.

```
artifact_path: modelflavors:  python_function:    data: model.xgb    env:      conda: conda.yaml      virtualenv: python_env.yaml    loader_module: mlflow.xgboost    python_version: 3.11.4  xgboost:    code: null    data: model.xgb    model_class: xgboost.sklearn.XGBClassifier    model_format: xgb    xgb_version: 2.0.3mlflow_version: 2.14.2model_size_bytes: 35040model_uuid: 516954aae7c94e91adeed9df76cb4052run_id: bf212703d1874eee9dcb37c8a92a6de6saved_input_example_info:  artifact_path: input_example.json  pandas_orient: split  type: dataframesignature:  inputs: '[{"type": "double", "name": "mean radius", "required": true}, {"type":    "double", "name": "mean texture", "required": true}, {"type": "double", "name":    "mean perimeter", "required": true}, {"type": "double", "name": "mean area", "required":    true}, {"type": "double", "name": "mean smoothness", "required": true}, {"type":    "double", "name": "mean compactness", "required": true}, {"type": "double", "name":    "mean concavity", "required": true}, {"type": "double", "name": "mean concave    points", "required": true}, {"type": "double", "name": "mean symmetry", "required":    true}, {"type": "double", "name": "mean fractal dimension", "required": true},    {"type": "double", "name": "radius error", "required": true}, {"type": "double",    "name": "texture error", "required": true}, {"type": "double", "name": "perimeter    error", "required": true}, {"type": "double", "name": "area error", "required":    true}, {"type": "double", "name": "smoothness error", "required": true}, {"type":    "double", "name": "compactness error", "required": true}, {"type": "double", "name":    "concavity error", "required": true}, {"type": "double", "name": "concave points    error", "required": true}, {"type": "double", "name": "symmetry error", "required":    true}, {"type": "double", "name": "fractal dimension error", "required": true},    {"type": "double", "name": "worst radius", "required": true}, {"type": "double",    "name": "worst texture", "required": true}, {"type": "double", "name": "worst    perimeter", "required": true}, {"type": "double", "name": "worst area", "required":    true}, {"type": "double", "name": "worst smoothness", "required": true}, {"type":    "double", "name": "worst compactness", "required": true}, {"type": "double", "name":    "worst concavity", "required": true}, {"type": "double", "name": "worst concave    points", "required": true}, {"type": "double", "name": "worst symmetry", "required":    true}, {"type": "double", "name": "worst fractal dimension", "required": true},    {"type": "double", "name": "additional_feature", "required": true}]'  outputs: '[{"type": "long", "required": true}]'  params: nullutc_time_created: '2024-07-30 08:38:11.745511'
```

Файл `MLmodel` содержит информациб по оберткам/вкусам (flavours), здесь это pyfunc и xgboost. Также он включает настройки окружения для conda (conda.yaml) и virtualenv (python_env.yaml). Модель является классификатором XGBoost, совместимым с API sklearn, сохранённым в формате XGBoost версии 2.0.3. Файл отслеживает такие детали, как размер модели, UUID, run ID и время создания. Также есть пример входных данных, для модели и её сигнатуроа - спецификация входа и выхода. Сигнатура может быть создана и сохранена вручную вместе с моделью, но MLflow автоматически генерирует сигнатуру при предоставлении примера входа.

> В экосистеме MLflow, flavors (вкусы) — это обертки для конкретных библиотек машинного обучения, которые позволяют сохранять, логировать и извлекать модели в едином формате. Это обеспечивает единообразное поведение метода предсказания (predict) для различных фреймворков, упрощая управление и развертывание моделей.

> Добавление предсказаний для примера входных данных может быть полезным, так как позволяет сразу протестировать модель после её загрузки и убедиться в её корректной работе в различных окружениях.

Мы сохранили две версии модели, обе с использованием xgboost, каждая из которых имеет два "flavors": `python_function` и `xgboost`. Разница заключается в классе модели: для бустера это `xgboost.core.Booster`, а для модели это `xgboost.sklearn.XGBClassifier`, который поддерживает API, совместимый с scikit-learn. Эти различия влияют на работу метода `predict`, поэтому важно просмотреть файл MLmodel и проверить сигнатуру модели перед использованием. Также могут быть небольшие различия в производительности `python_function` обычно немного медленее.

При загрузке модели бустера с использованием xgboost, модель ожидает, что входные данные будут в форме объекта DMatrix, и метод predict в нашем случае будет выдавать оценки (а не классы).

#### Built-in evaluation

Встроенная функция `mlflow.evaluate` позволяет оценивать модели на дополнительных наборах данных:

```
import sysimport osimport argparseimport warningsimport loggingimport mlflowimport pandas as pdfrom loguru import loggerlogger.remove()logger.add(sys.stdout, format="{time:YYYY-MM-DD HH:mm:ss} | {level} | {message}")warnings.filterwarnings('ignore')logging.getLogger('mlflow').setLevel(logging.ERROR)if __name__ == '__main__':    logger.info('Evaluation started')    parser = argparse.ArgumentParser()    parser.add_argument("--eval-dataset", type=str)    eval_dataset = pd.read_csv(parser.parse_args().eval_dataset)            with mlflow.start_run() as run:                eval_dataset = mlflow.data.from_pandas(            eval_dataset, targets="target"        )        last_version = mlflow.MlflowClient().get_registered_model('CancerModel').latest_versions[0].version        mlflow.evaluate(            data=eval_dataset, model_type="classifier", model=f'models:/CancerModel/{last_version}'        )        logger.success('Evaluation finished')
```

```
mlflow run . --entry-point data-evaluation --env-manager conda --experiment-name Cancer_Classification --run-name Data_Evaluation -P eval-dataset='test.csv'
```

Результаты можно просмотреть в интерфейсе MLflow, где представлены различные метрики и графики, включая ROC-AUC, матрицы ошибок и графики SHAP (если SHAP установлен).

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/b81/754/46f/b8175446fba57dbc66d4939e8981912d.png)

### Model Serving

MLflow имеет встроенные возможности для деплоя моделей. Деплой модели с использованием Flask довольно просто сделать: `mlflow models serve -m models:/CancerModel/1 --env-manager local`. Но мы также можем использовать `mlserver`.

> Пакет `mlserver` облегчает эффективное развертывание и обслуживание моделей машинного обучения с поддержкой множества фреймворков, используя интерфейсы REST и gRPC. Он интегрируется с Seldon Core для масштабируемого и надёжного управления моделями и мониторинга.

Возможно, вам потребуется установить следующие пакеты: `mlserver`, `mlserver-mlflow`, `mlserver-xgboost`, если вы используете собственное окружение. После этого мы можем настроить конфигурационный файл (model-settings.json) для MLServer. Это позволяет нам изменить работу API; здесь мы просто настраиваем алиас для модели:

```
{    "name": "cancer-model",    "implementation": "mlserver_mlflow.MLflowRuntime",    "parameters": {        "uri": "models:/CancerModel/1"    }}
```

Чтобы запустить MLServer в локальном окружении, можно использовать команду `mlserver start .`

> Теперь у нас есть рабочий API с документацией OpenAPI, валидацией запросов, HTTP и gRPC серверами и эндпоинтом с метриками для prometheus. И всё это без написания кода, все что нужноя - простая конфигурацию в формате JSON.

Мы можем проверить документацию для нашей модели и изучить ожидаемую структуру данных через Swagger по адресу `/v2/models/cancer/model/docs`

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/04e/855/c4a/04e855c4a3e45c29c089b87c582b685b.png)

Мы можем получить доступ к эндпоинту с метриками или настроить prometheus для их сбора

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/a52/892/ad3/a52892ad33d8342e424a178e4f020ccb.png)

Теперь можно отправлять запросы к модели:

```
import requestsimport jsonurl = "http://127.0.0.1:8080/invocations"# convert df do split format and then to jsoninput_data = json.dumps({    "params": {      'method': 'proba',      },    'dataframe_split': {        "columns": test.columns.tolist(),        "data": test.values.tolist()    }})# Send a POST request to the MLflow model serverresponse = requests.post(url, data=input_data, headers={"Content-Type": "application/json"})if response.status_code == 200:    prediction = response.json()    print("Prediction:", prediction)else:    print("Error:", response.status_code, response.text)
```

```
Prediction: {'predictions': [1, 1, 1, 0, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 1, 0, 0, 1, 1, 1, 1, 0, 0, 0, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 1, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 0, 0, 1, 0, 1, 1, 1, 1, 0, 1, 0, 1, 1, 1, 1, 1, 0, 0, 1, 0, 1, 1, 1, 0, 1, 1, 1, 1, 1, 0, 0, 1, 0, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 0, 0, 1, 0, 0, 0, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 0, 0, 0, 1, 0, 0, 1, 1, 0, 1, 0, 1, 0, 1, 1, 1, 1, 0, 1, 0, 1, 0, 1, 0, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 0, 1, 1, 1, 0, 1, 0, 0, 0, 1, 1, 0, 1, 0, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 1, 1, 1, 1, 1, 0, 1, 0, 0, 0, 1, 1, 1, 0, 1, 1]}
```

#### Customize model

Мы можем настроить нашу модель для предоставления вероятностей или включения дополнительного логирования. Для этого сначала скачаем модель, а затем обернем её в пользовательскую обертку:

```
import mlflowimport mlflow.xgboostimport mlflow.pyfunc# Step 1: Download the Existing Model from MLflowmodel_uri = "models:/CancerModel/1"model = mlflow.xgboost.load_model(model_uri)# Step 2: Define the Custom PyFunc Model with `loguru` Setup in `load_context`class CustomPyFuncModel(mlflow.pyfunc.PythonModel):        def __init__(self, model):        self.model = model            def get_logger(self):        from loguru import logger        logger.remove()        logger.add("mlserve/mlserver_logs.log", format="{time:YYYY-MM-DD HH:mm:ss} | {level} | {message}")        return logger            def load_context(self, context):        self.logger = self.get_logger()    def predict(self, context, model_input):                self.logger.info(f"start request")        self.logger.info(f"batch size: {len(model_input)}")                predict =  self.model.predict_proba(model_input)[:,1]                self.logger.success(f"Finish request")                return predict        # Step 3: Save the Wrapped Model Back to MLflowwith mlflow.start_run() as run:    mlflow.pyfunc.log_model(        artifact_path="custom_model",        python_model=CustomPyFuncModel(model),        registered_model_name="CustomCancerModel",    )
```

Теперь перепишем конфигурационный файл и заново запустим mlserver:

```
{    "name": "cancer-model",    "implementation": "mlserver_mlflow.MLflowRuntime",    "parameters": {        "uri": "models:/CustomCancerModel/1"    }}
```

Отправим запросы к новому API:

```
# Send a POST request to the MLflow model serverresponse = requests.post(url, data=input_data, headers={"Content-Type": "application/json"})if response.status_code == 200:    prediction = response.json()    print("Prediction:", prediction['predictions'][:10])else:    print("Error:", response.status_code, response.text)
```

```
Prediction: [0.9406537413597107, 0.9998677968978882, 0.9991995692253113, 0.00043031785753555596, 0.9973010420799255, 0.9998010993003845, 0.9995433688163757, 0.9998323917388916, 0.0019207964651286602, 0.0004339178267400712]
```

Теперь мы получили, веротяности, также можно проверить что дополнительные логи записаны в файл:

```
2024-07-30 12:02:38 | INFO | start request2024-07-30 12:02:38 | INFO | batch size: 1882024-07-30 12:02:38 | SUCCESS | Finish request
```

> Альтернативой этому подходу является написание нового API с нуля. Это может быть лучше, когда нам нужна большая гибкость, дополнительная функциональность или когда мы ограничены во времени, так как использование MLServer может быть немного медленнее.

> В случае периодического переобучения модели нам нужно обновлять модель в сервисе или перезапускать деплой в открытой версии MLflow я не нашел такого функционала. Версия Databricks включает функцию вебхуков, которая позволяет MLflow уведомлять API о новых версиях. Другим вариантом является запуск деплоя при обновлении модели. Мы также можем открыть дополнительный эндпоинт в сервере и вызывать его в рамках DAG, или сконфигурирвовать сервер периодически запрашивать обновления в MLflow.

### MLflow Tracking Server

#### MLflow Local Setup

До этого мы работали с MLflow локально: метаданные и артефакты хранятся в папке по умолчанию `mlruns`. Вы можете проверить эту папку у себя, если успешно выполнили все предыдущие шаги. Мы можем изменить место хранения метаданных MLflow, указав другой `backend-store-uri` при запуске команды MLflow UI. Например, чтобы использовать другую папку (`mlruns_new`), выполните следующие действия: `mlflow ui --backend-store-uri ./mlruns_new` И поменяйте tracking uri в проекте: `mlflow.set_tracking_uri("file:./mlruns_new")`.

#### Remote Tracking

В продакшене мы обычно настраиваем удалённый сервер отслеживания с хранилищем артефактов и базой данных для метаданных MLflow. Мы можем симулировать эту конфигурацию, используя MinIO для хранения артефактов и PostgreSQL для базы данных. Вот простой файл Docker Compose с 4 сервсиами:

1. MinIO как s3 подобное хранилище
    
2. Клиент MinIO (minio/mc) для создания бакета для MLflow
    
3. PostgreSQL в качестве базы данных для метаинформации
    
4. MLflow UI
    

```
services:  s3:    image: minio/minio    container_name: mlflow_s3    ports:      - 9000:9000      - 9001:9001    command: server /data --console-address ':9001'    environment:      - MINIO_ROOT_USER=mlflow      - MINIO_ROOT_PASSWORD=password    volumes:      - minio_data:/data  init_s3:    image: minio/mc    depends_on:      - s3    entrypoint: >      /bin/sh -c "      until (/usr/bin/mc alias set minio http://s3:9000 mlflow password) do echo '...waiting...' && sleep 1; done;      /usr/bin/mc mb minio/mlflow;      exit 0;      "  postgres:    image: postgres:latest    ports:      - 5432:5432    environment:      - POSTGRES_USER=mlflow      - POSTGRES_PASSWORD=password    volumes:      - postgres_data:/var/lib/postgresql/data  mlflow:    build:      context: .      dockerfile: Dockerfile    ports:      - 5050:5000    environment:      - MLFLOW_S3_ENDPOINT_URL=http://s3:9000      - AWS_ACCESS_KEY_ID=mlflow      - AWS_SECRET_ACCESS_KEY=password    command: >      mlflow server        --backend-store-uri postgresql://mlflow:password@postgres:5432/mlflow        --default-artifact-root s3://mlflow/        --artifacts-destination s3://mlflow/        --host 0.0.0.0    depends_on:      - s3      - postgres      - init_s3volumes:  postgres_data:  minio_data:
```

Вызовете эти команды для создания имэджа и запуска контенеров `docker compose -f tracking_server/docker-compose.yml build` and `docker compose -f tracking_server/docker-compose.yml up`.

> Вместо обработки артефактов MLflow предоставляет ссылку клиенту, позволяя сохранять и загружать артефакты напрямую из хранилища артефактов. Поэтому вам нужно настроить ключи доступа и хост сервера отслеживания, чтобы правильно логировать артефакты.

```
import osimport mlflowos.environ['AWS_ACCESS_KEY_ID'] = 'mlflow'os.environ['AWS_SECRET_ACCESS_KEY'] = 'password'os.environ['MLFLOW_TRACKING_URI'] = 'http://localhost:5050'os.environ['MLFLOW_S3_ENDPOINT_URL'] = 'http://localhost:9000'# run data preprocessing step one more timemlflow.run(    uri = '.',    entry_point = 'data-preprocessing',    env_manager='local',    experiment_name='Cancer_Classification',    run_name='Data_Preprocessing',    parameters={'test-size': 0.5},)
```

После этого шага мы можем зайти в MLflow UI, чтобы убедиться, что запуск и артефакты были успешно зарегистрированы:

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/dbf/ab2/990/dbfab29908330c197ae15d2ac31ee3c0.png)

И убедится через интерфейс MinIO, что артефакты были успешно сохранены в бакете:

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/579/ae5/e7d/579ae5e7dd2fc5933a1c8f1a692d8898.png)

И также выполнить запрос к нашей базе данных PostgreSQL, чтобы убедиться, что она используется для хранения метаданных:

```
import psycopg2import pandas as pdconn = psycopg2.connect(dbname='mlflow', user='mlflow', password='password', host='localhost', port='5432')try:    query = "SELECT experiment_id, name FROM experiments"    experiments_df = pd.read_sql(query, conn)except Exception as e:    print(e)else:    print(experiments_df)finally:    conn.close()
```

> В продакшене может потребоваться аутентификация, разные уровни доступа пользователей и мониторинг для MLflow, бакета, базы данных и конкретных артефактов. Мы не будем охватывать эти аспекты здесь, и некоторые из них не имеют встроенной поддержки в MLflow.

Мы рассмотрели как можно использовать MLflow как локально для себя, так и для совасместной работы над проектами, кратко прошлись по основному функционалу и использовали его на примерах. Вы можете посмотреть полный код в этом [репозитории](https://github.com/egorborisov/mlflow_example), также можно посомтреть [документацию Mlflow](https://mlflow.org/docs/latest/index.html).