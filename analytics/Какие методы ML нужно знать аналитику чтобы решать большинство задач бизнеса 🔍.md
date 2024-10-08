---
tags:
  - data
author: Balandin
data_type:
  - Analytics
link: https://t.me/suranalytics/181
source: telegram
company: Aviasales
---
 
Сейчас все говорят про LLM, Deep Learning и AGI. Но если вы только начинаете, то сразу кидаться изучать это точно не стоит. 
Но вообще, надо определиться с целями. Мой пост для тех, кто либо только погружается, либо уже преисполнился в аналитике и хочет начать решать некоторые новые задачи с применением машинного обучения. Хотя я уверен, что несмотря на крутое развитие области, во многих компаниях все ещё крутятся простые линейные модели и тем не менее приносят пользу. А мы как аналитики как раз этим и занимаемся - помогаем бизнесу расти. 

Но информации, курсов, книг очень много, и не очень понятно с чего начинать. Предлагаю вооружиться принципом Парето (что 20% усилий дают 80% результата) и начать вот с этих методов: 

- Регрессия. Предсказываем непрерывное значение. Применений масса - начиная от предсказания LTV, спроса, заканчивая динамическим ценообразованием.  
- Классификация. Предсказываем класс объекта. Используется в большинстве продуктов - предсказание оттока клиентов, выделение спама, вероятности не вернуть кредит. 
- Кластеризация. Группировка похожих объектов между собой. Используется для сегментации клиентов, поиска похожих паттернов в поведении.  
- Работа с временными рядами. Так же востребовано во многих сферах - прогноз продаж, основных метрик для алертинга, часто используется для финансового планирования.  

Если опыта совсем нет, то рекомендую вот эти курсы на kaggle. Это отличное быстрое введение, где можно сразу начать решать небольшие задачи и потрогать ML на практике. Все-таки без понимания основных концептов про работу с данными, метрики, feature engineering, валидацию моделей, подбор гиперпараметров не стоит кидаться бездумно делать fit_predict из условного XGBoost. Поэтому вот:

- Введение в ML (https://www.kaggle.com/learn/intro-to-machine-learning)
- Intermediate ML (https://www.kaggle.com/learn/intermediate-machine-learning)
- Feature engineering (https://www.kaggle.com/learn/feature-engineering)
- Временные ряды (https://www.kaggle.com/learn/time-series)

А вот огромный роадмап (https://github.com/mrdbourke/machine-learning-roadmap?tab=readme-ov-file) и подборка ресурсов по ML. Можно двигаться точечно и искать материалы по мере необходимости, ну или закопаться с головой и пройти вообще все. 

Когда база получена, можно постепенно начинать решать свои задачи на работе. А чтобы чуть проще было разобраться с таким огромным кол-вом всего, предлагаю обратить внимание на auto-ml библиотеку PyCaret, где можно предобработать данные, перебрать много моделей и гиперпараметров и получить good enough решение. Ну и всякие дополнительные плюшки в виде интерпретируемости моделей и логирования экспериментов тоже есть. 

Примеры (https://pycaret.gitbook.io/docs/get-started/tutorials), со всеми методами, про которые писал выше.