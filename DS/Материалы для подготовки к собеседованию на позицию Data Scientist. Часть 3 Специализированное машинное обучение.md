---
tags:
  - data
author: Ryblov
link: https://habr.com/ru/companies/megafon/articles/808585/
source: habr
data_type:
  - DS
company: Megafon
---



Привет! Меня зовут Артем. Я работаю Data Scientist'ом в компании МегаФон (платформа для безопасной монетизации данных [OneFactor](https://nn.megafon.ru/onefactor/)). Мы строим скоринговые (credit scoring), лидогенерационные (lead generation) и антифрод (anti-fraud) модели на телеком данных, а также делаем гео-аналитику (geo-analytics).

В [предыдущей статье](https://habr.com/ru/companies/megafon/articles/800919/) я поделился материалами для подготовки к этапу по классическому машинному обучению.

Давайте вспомним из каких секций состоит процесс собеседований на позицию Data Scientist:

- [Live Coding](https://habr.com/ru/companies/megafon/articles/795261/)
    
- [Классическое машинное обучение](https://habr.com/ru/companies/megafon/articles/800919/)
    
- **Специализированное машинное/глубокое обучение**
    
- [Дизайн систем машинного обучения](https://habr.com/ru/companies/megafon/articles/821557/) (middle+/senior)
    
- Поведенческое интервью (middle+/senior)
    

![Типичный процесс интервью глазами Dall-E 3](https://habrastorage.org/r/w1560/webt/w8/ms/ym/w8msymoreolitujki3i3guxhdlm.jpeg)

Типичный процесс интервью глазами Dall-E 3

В этой статье рассмотрим материалы, которые можно использовать для подготовки к секции по специализированному машинному обучению.

Замечания

## Содержание

- [Материалы](https://habr.com/ru/companies/megafon/articles/808585/#%D0%BC%D0%B0%D1%82%D0%B5%D1%80%D0%B8%D0%B0%D0%BB%D1%8B)
    
    - [Глубокое обучение](https://habr.com/ru/companies/megafon/articles/808585/#%D0%B3%D0%BB%D1%83%D0%B1%D0%BE%D0%BA%D0%BE%D0%B5-%D0%BE%D0%B1%D1%83%D1%87%D0%B5%D0%BD%D0%B8%D0%B5)
        
    - [Обработка текстов на естественном языке](https://habr.com/ru/companies/megafon/articles/808585/#%D0%BE%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B0-%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%BE%D0%B2-%D0%BD%D0%B0-%D0%B5%D1%81%D1%82%D0%B5%D1%81%D1%82%D0%B2%D0%B5%D0%BD%D0%BD%D0%BE%D0%BC-%D1%8F%D0%B7%D1%8B%D0%BA%D0%B5)
        
    - [Компьютерное зрение](https://habr.com/ru/companies/megafon/articles/808585/#%D0%BA%D0%BE%D0%BC%D0%BF%D1%8C%D1%8E%D1%82%D0%B5%D1%80%D0%BD%D0%BE%D0%B5-%D0%B7%D1%80%D0%B5%D0%BD%D0%B8%D0%B5)
        
    - [Графовые нейронные сети](https://habr.com/ru/companies/megafon/articles/808585/#%D0%B3%D1%80%D0%B0%D1%84%D0%BE%D0%B2%D1%8B%D0%B5-%D0%BD%D0%B5%D0%B9%D1%80%D0%BE%D0%BD%D0%BD%D1%8B%D0%B5-%D1%81%D0%B5%D1%82%D0%B8)
        
    - [Обучение с подкреплением](https://habr.com/ru/companies/megafon/articles/808585/#%D0%BE%D0%B1%D1%83%D1%87%D0%B5%D0%BD%D0%B8%D0%B5-%D1%81-%D0%BF%D0%BE%D0%B4%D0%BA%D1%80%D0%B5%D0%BF%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5%D0%BC)
        
    - [Рекомендательные системы](https://habr.com/ru/companies/megafon/articles/808585/#%D1%80%D0%B5%D0%BA%D0%BE%D0%BC%D0%B5%D0%BD%D0%B4%D0%B0%D1%82%D0%B5%D0%BB%D1%8C%D0%BD%D1%8B%D0%B5-%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D1%8B)
        
    - [Временные ряды](https://habr.com/ru/companies/megafon/articles/808585/#%D0%B2%D1%80%D0%B5%D0%BC%D0%B5%D0%BD%D0%BD%D1%8B%D0%B5-%D1%80%D1%8F%D0%B4%D1%8B)
        
    - [Big Data](https://habr.com/ru/companies/megafon/articles/808585/#big-data)
        
- [Подведем итоги](https://habr.com/ru/companies/megafon/articles/808585/#%D0%BF%D0%BE%D0%B4%D0%B2%D0%B5%D0%B4%D0%B5%D0%BC-%D0%B8%D1%82%D0%BE%D0%B3%D0%B8)
    
- [Что дальше?](https://habr.com/ru/companies/megafon/articles/808585/#%D1%87%D1%82%D0%BE-%D0%B4%D0%B0%D0%BB%D1%8C%D1%88%D0%B5)
    

## Материалы

Кроме основ машинного обучения (полезные материалы мы рассмотрели в [предыдущей статье](https://habr.com/ru/companies/megafon/articles/800919/)), во многих задачах требуются также специфические знания, которые у вас наверняка проверят в рамках одного из этапов собеседования.

Давайте перечислим самые популярные направления ML в данный момент:

- Обработка естественного языка (Natural Language Processing / NLP)
    
- Компьютерное зрение (Computer Vision / CV)
    
- Распознавание речи (Automated Speech Recognition (ASR) / speech-to-text (STT))
    
- Синтез речи (Speech Synthesis / text-to-speech (TTS))
    
- Обучение с подкреплением (Reinforcement Learning / RL)
    
- Временные ряды (Time Series / TS)
    
- Рекомендательные системы (Recommender System / RecSys)
    

Во всех этих направлениях в данный момент активно используются методы глубокого обучения (Deep Learning / DL), хотя изначально, во многих из них использовались классические ML подходы над вручную созданными признаками (hand-crafted features). Поэтому сначала мы рассмотрим материалы для изучения теории глубокого обучения, а затем погрузимся в более прикладные области.

### Глубокое обучение

Нейросетевые подходы сейчас доминируют в решении задач, где требуется анализ неструктурированных данных (текст, речь, видео, изображение). Ниже рассмотрим материалы, которые помогут в изучении теории и практики глубокого обучения.

#### Книги

⭐ Шолле Ф. [**Глубокое обучение на Python**](https://www.piter.com/collection/all/product/glubokoe-obuchenie-na-python-2-e-mezhd-izdanie) **/** [**Deep Learning with Python**](https://www.manning.com/books/deep-learning-with-python-second-edition) by François Chollet

![Мое первое издание (а вам лучше выбрать второе)](https://habrastorage.org/r/w1560/webt/_p/qb/3k/_pqb3km854wnhsibettwm9dwlvw.jpeg)

Мое первое издание (а вам лучше выбрать второе)

Отличная книга, которая послужит идеальным введением в глубокое обучение. Кстати, первое издание этой книги есть у меня в печатном формате и пригодилось мне не раз, когда я только начинал заниматься DL.

⭐ [**Dive into Deep Learning**](https://d2l.ai/index.html)

![Dive into Deep Learning](https://habrastorage.org/r/w1560/getpro/habr/upload_files/d53/ef2/ce9/d53ef2ce988aa4244243c236918e4f89.png)

Dive into Deep Learning

Интерактивная книга по глубокому обучению с кодом (на PyTorch, NumPy/MXNet, JAX и TensorFlow) и математикой. Каждая секция в книге является отдельным Jupyter notebook'ом, код в котором можно запускать и менять по своему усмотрению. Используется в 500 университетах в 70 странах мира.

[**Understanding Deep Learning**](https://udlbook.github.io/udlbook/) by Simon J.D. Prince

![Understanding Deep Learning](https://habrastorage.org/r/w1560/getpro/habr/upload_files/751/d96/d46/751d96d46efe9fe652e7830fe67f0a5e.jpeg)

Understanding Deep Learning

Эта книга посвящена идеям, лежащим в основе глубокого обучения. В первой части книги представлены модели глубокого обучения и обсуждается, как их обучать, измерять их производительность и улучшать ее. В следующей части рассматриваются архитектуры для работы с изображениями, текстом и графами. Эти главы требуют только вводных знаний по линейной алгебре, математическому анализу и теории вероятностей и должны быть доступны любому студенту второго курса технического университета. Последующие части книги посвящены генеративным моделям и обучению с подкреплением. Эти главы требуют дополнительных знаний в области вероятностей и математического анализа, и предназначены для более продвинутых студентов.

[**The Little Book of Deep Learning**](https://fleuret.org/public/lbdl.pdf) by François Fleuret

![Схематическое изображение нейронной сети](https://habrastorage.org/r/w1560/webt/-s/vk/dk/-svkdk1qvtjjifgmed0gylv9dwg.png)

Схематическое изображение нейронной сети

Здесь кратко (но содержательно) рассматриваются многие важные темы: метод обратного распространения ошибки, дропаут, нормализация, функции активации, слои внимания и тд. При этом, книга подойдет даже новичкам: в начале автор раскрывает базовые понятия, и даже рассказывает про GPU и тензоры. Стоит также отметить, что данная версия адаптирована под чтение на телефоне.

⭐ [**What are embeddings**](https://vickiboykis.com/what_are_embeddings/index.html) by Vicki Boykis

![Идея эмбеддингов в одной картинке](https://habrastorage.org/r/w1560/webt/go/7q/go/go7qgoed2gx0filduexjlj4v3b4.png)

Идея эмбеддингов в одной картинке

Данная книга посвящена эмбеддингам:

- Объясняется что такое эмбеддинги (на примере рекомендательной системы)
    
- Рассматриваются классические и современные подходы к созданию эмбеддингам
    
- Обсуждается использование эмбеддингов в production (как их получать, хранить, оценивать и тд)
    

#### Курсы

⭐ [**Школа глубокого обучения**](https://dls.samcs.ru/) ФПМИ МФТИ

![Deep Learning School](https://habrastorage.org/r/w1560/webt/df/cu/fr/dfcufrhsozk72nv5qltgwduec10.png)

Deep Learning School

Школа глубокого обучения — это образовательный проект Физтех-школы прикладной математики и информатики МФТИ. Обучение нейросетям происходит с самых основ и до продвинутого уровня. Занятия ведут выпускники ФПМИ МФТИ, имеющие опыт разработки и исследований в области AI.

Программа 1-ой части курса (Введение в ML, DL и CV):

- Введение в искусственный интеллект
    
- Основы машинного обучения
    
- Линейные модели
    
- Композиции алгоритмов МО и выбор модели
    
- Введение в нейронные сети
    
- Сверточные нейросети (CNN)
    
- Продвинутое обучение нейросетей
    
- Архитектуры CNN и Transfer Learning
    
- Семантическая сегментация
    
- Детекция объектов на изображении
    
- Автоэнкодеры
    
- Генеративное-состязательные сети (GAN)
    
- Итоговый проект
    

Программа 2-ой части курса (NLP, Audio):

- Введение в NLP. Эмбеддинги слов
    
- Рекуррентные нейросети (RNN)
    
- Языковое моделирование
    
- Машинный перевод, архитектура encoder-decoder
    
- Механизм Attention и архитектура Transformer
    
- Предобучение и дообучение языковых моделей
    
- GPT, GPT-2, GPT-3, ChatGPT, Zero-shot Learning
    
- Задачи NLP и методы их решения
    
- Суммаризация текста и диалоговые системы
    
- Дистилляция больших моделей
    
- Введение в обработку аудио
    
- Введение в распознавание речи
    
- Итоговый проект
    

[**Deep Learning Specialization**](https://www.coursera.org/specializations/deep-learning) `paid`

![Сертификат об окончании специализации Deep Learning](https://habrastorage.org/r/w1560/webt/46/yq/kc/46yqkcqk2iubyqrl67gmnpet61g.png)

Сертификат об окончании специализации Deep Learning

Хорошая специализация по глубокому обучению на Coursera. Подойдет, как вводный курс, но на момент ее прохождения (2019 год), практика не была сильной стороной этой специализации (можете почитать отзывы).  
Я бы порекомендовал посмотреть [лекции](https://www.youtube.com/@Deeplearningai/playlists) и не платить за нее.

[**MIT 6.S191. Introduction to Deep Learning**](http://introtodeeplearning.com/)

![Список лекций курса MIT 6.S191 Introduction to Deep Learning](https://habrastorage.org/r/w1560/webt/hz/sw/do/hzswdonrwjzuzk5pgknvatmo0ym.png)

Список лекций курса MIT 6.S191 Introduction to Deep Learning

Вводная программа Массачусетского технологического института по методам глубокого обучения с применением к обработке естественного языка, компьютерному зрению, биологии и многому другому! На ней вы узнаете об алгоритмах глубокого обучения, получите практический опыт построения нейронных сетей и освоите новейшие темы, включая большие языковые модели и генеративный искусственный интеллект.

⭐ [**Practical Deep Learning for Coders от fast.ai**](https://course.fast.ai/)

![Deep Learning for Coders with Fastai and PyTorch: AI Applications Without a PhD](https://habrastorage.org/r/w1560/getpro/habr/upload_files/f30/f05/efc/f30f05efc348a6a71706bc1e813141f9.jpg)

Deep Learning for Coders with Fastai and PyTorch: AI Applications Without a PhD

Бесплатный курс, предназначенный для людей, которые хотят научиться применять глубокое обучение и машинное обучение для решения практических задач:

- Обучение DL моделей для компьютерного зрения, обработки естественного языка, решения табличных задач и задач коллаборативной фильтрации
    
- Развертывание моделей
    
- Использование PyTorch, а также популярных библиотек, таких как fastai и Hugging Face
    

Отлично дополняет курс бесплатная книга [Deep Learning for Coders with Fastai and PyTorch: AI Applications Without a PhD](https://course.fast.ai/Resources/book.html).

⭐ [**Learn PyTorch for Deep Learning: Zero to Mastery**](https://www.learnpytorch.io/)

![Содержание курса Learn PyTorch for Deep Learning: Zero to Mastery](https://habrastorage.org/r/w1560/webt/n5/d5/v4/n5d5v4jjxyhnjyfwqtb0t44q1ri.jpeg)

Содержание курса Learn PyTorch for Deep Learning: Zero to Mastery

Этот курс научит вас основам машинного обучения и глубокого обучения с помощью PyTorch:

- Fundamentals
    
- Neural Network Classification
    
- Computer Vision
    
- Custom Datasets
    
- Going Modular
    
- Transfer Learning
    
- Experiment Tracking
    
- Paper Replicating
    
- Model Deployment
    

[**Full Stack Deep Learning**](https://fullstackdeeplearning.com/course/2022/)

![Идея курса Full Stack Deep Learning](https://habrastorage.org/r/w1560/webt/95/sq/lk/95sqlkwtupwaptgfrps48jrxuye.png)

Идея курса Full Stack Deep Learning

Курс подойдет людям, которые уже знают основы DL и хотели бы лучше разобраться, как применять DL в реальных задачах:

- Постановка задачи и оценка стоимости проекта
    
- Поиск, очистка, обработка, разметка, синтез и аугментация данных
    
- Выбор подходящей платформы и вычислительной инфраструктуры
    
- Устранение неполадок в обучении и обеспечение воспроизводимости результатов
    
- Развертывание модели в production
    
- Мониторинг и постоянное улучшение развернутой модели
    
- Как работают ML-команды и как управлять ML-проектами
    
- Использование больших языковых моделей (LLM) и других Foundation моделей
    

[**Efficient Deep Learning Systems**](https://github.com/mryab/efficient-dl-systems)

![Efficient Deep Learning Systems GitHub](https://habrastorage.org/r/w1560/webt/hm/vn/ok/hmvnokzvuzbsqzakcjuglqu-6ry.png)

Efficient Deep Learning Systems GitHub

Данный курс, как и предыдущий, больше направлен на продукционализацию и оптимизацию DL моделей:

- Introduction
    
- Experiment tracking, model and data versioning, testing DL code in Python
    
- Training optimizations, profiling DL code
    
- Basics of distributed ML
    
- Data-parallel training and All-Reduce
    
- Training large models
    
- Python web application deployment
    
- LLM inference optimizations and software
    
- Efficient model inference
    

⭐ [**Neural Networks: Zero to Hero**](https://karpathy.ai/zero-to-hero.html) by Andrej Karpathy

![Андрей Карпати размышляет о том, что такое backpropagation](https://habrastorage.org/r/w1560/webt/ax/bk/qg/axbkqgyqy4tefacc0n0y9-eziyk.jpeg)

Андрей Карпати размышляет о том, что такое backpropagation

Курс Андрея Карпати по построению нейронных сетей с нуля - с основ обратного распространения ошибки до современных глубоких нейронных сетей, таких как GPT.  
По мнению автора, языковые модели — отличное место для изучения глубокого обучения, даже если вы намерены в конечном итоге перейти к другим областям, таким как компьютерное зрение, потому что большую часть того, что вы изучите, можно будет сразу же применить и в новой для вас области.

#### Шпаргалки

![Шпаргалки по DL от Афшин и Шервин Амиди](https://habrastorage.org/r/w1560/webt/dy/cs/ks/dycsksb6qlfqtgbnxxaowuxlsnc.png)

Шпаргалки по DL от Афшин и Шервин Амиди

Братья Афшин и Шервин Амиди подготовили гайды, в которых подчеркиваются важные моменты каждого предмета, которые Шервин преподавал в Стэнфорде.

Данные материалы подойдут, когда нужно за короткое время, например, перед собеседованием, вспомнить основные моменты в определенной теме.

Не забывайте, что про похожие шпаргалки, но уже по классическому ML, я писал в [этом разделе](https://habr.com/ru/companies/megafon/articles/800919/#%D1%88%D0%BF%D0%B0%D1%80%D0%B3%D0%B0%D0%BB%D0%BA%D0%B8) второй статьи.

[**Convolutional Neural Networks**](https://stanford.edu/~shervine/teaching/cs-230/cheatsheet-convolutional-neural-networks)

- Типы слоев, гиперпараметры для фильтров, функции активации
    
- Обнаружение объектов (Object Detection), распознавание лиц (Face Verification and Recognition)
    
- Перенос нейростиля (Neural Style Transfer); архитектуры, использующие вычислительные трюки
    

[**Recurrent Neural Networks**](https://stanford.edu/~shervine/teaching/cs-230/cheatsheet-recurrent-neural-networks)

- Исчезающий/расширяющийся градиент (vanishing/exploding gradient), GRU, LSTM, разновидности RNNs
    
- Word2vec, skip-gram, негативное сэмплирование (negative sampling), GloVe, модель внимания (attention model)
    
- Языковая модель (language model), beam search, Bleu скор
    

[**Deep Learning Tips and Tricks**](https://stanford.edu/~shervine/teaching/cs-230/cheatsheet-deep-learning-tips-and-tricks)

- Аугментация данных (data augmentation), пакетная нормализация (batch normalization), регуляризация
    
- Инициализация Ксавье (Xavier initialization), трансферное обучение (transfer learning), адаптивная скорость обучения (adaptive learning rates)
    
- Переобучение на маленьких батчах (overfitting small batch), проверка градиента (gradient checking)
    

#### Разное

⭐ [**Deep Learning Models**](https://github.com/rasbt/deeplearning-models) by Sebastian Raschka

![Пример материалов, которые предлагает сайт](https://habrastorage.org/r/w1560/webt/ve/sm/ch/vesmchbpyuuuf7xrv_kzau8sdki.png)

Пример материалов, которые предлагает сайт

Коллекция различных архитектур глубокого обучения, моделей и советов для обучения (TensorFlow и PyTorch) в Jupyter Notebooks.

[**Deep Learning Tuning Playbook**](https://github.com/google-research/tuning_playbook)

![](https://habrastorage.org/r/w1560/webt/iy/ki/ld/iykildtqzapazxomgoqcv5rz8-c.png)

Руководство по настройке гиперпараметров для моделей глубокого обучения.

[**The Incredible PyTorch**](https://github.com/ritchieng/the-incredible-pytorch)

![Невероятный PyTorch](https://habrastorage.org/r/w1560/webt/bn/hw/1i/bnhw1izuqne-3buauhmdib3hfiq.png)

Невероятный PyTorch

Это тщательно подобранный список туториалов, проектов, библиотек, видео, статей, книг и всего, что связано с PyTorch.

### Обработка текстов на естественном языке

В данном разделе рассмотрим материалы (в основном курсы) по обработке текстов на естественном языке.

Материалы (40+ ссылок), целиком посвященные большим языковым моделям (Large Language Models / LLM) и Prompt Engineering не влезают в статью, поэтому их вы сможете найти в репозитории [Data Science Resources](https://github.com/Extremesarova/ds_resources).

#### Курсы

⭐ [**Курс по NLP**](https://github.com/yandexdataschool/nlp_course) от ШАД (Елена Войта)

![Содержание одного блока курса NLP Course | For You](https://habrastorage.org/r/w1560/webt/em/ad/mx/emadmxxeqyrfk5g7kvqawlwxnzy.png)

Содержание одного блока курса NLP Course | For You

Отличный курс, покрывающий одновременно и классические подходы, и современные темы:

1. Word Embeddings
    
2. Text Classification
    
3. Language Modeling
    
4. Seq2seq and Attention
    
5. Transfer Learning
    
6. LLMs and Prompting
    
7. Transformer architecture & training tricks
    
8. Conversation systems, instruction fine-tuning & RLHF
    
9. Efficient inference of NLP models
    

В дополнение к курсу [Елена Войта](https://lena-voita.github.io/) (автор) подготовила замечательные интерактивные материалы [NLP Course | For You](https://lena-voita.github.io/nlp_course.html).

[**Нейронные сети и обработка текста**](https://stepik.org/course/54098/info)

![Один из авторов курса - Роман Суворов](https://habrastorage.org/r/w1560/getpro/habr/upload_files/05e/dd6/7ea/05edd67eaf725c11a31f2f57bd523d03.jpg "Один из авторов курса - Роман Суворов")

Один из авторов курса - Роман Суворов

Давно не обновляемый (2019 год), но все еще хороший курс для изучения основ NLP от специалистов Samsung AI Center:

1. Введение
    
2. Векторная модель текста и классификация длинных текстов
    
3. Базовые нейросетевые методы работы с текстами
    
4. Языковые модели и генерация текста
    
5. Преобразование последовательностей: 1-к-1 и N-к-M
    
6. Transfer learning, адаптация моделей
    
7. Финальное соревнование на kaggle и заключение
    

[**Курс по NLP**](https://ods.ai/tracks/nlp-course-spring-2024) от Валентина Малых

![](https://habrastorage.org/r/w1560/webt/_c/x-/k0/_cx-k0lz_rvw3x8lyecitmlfih0.png)

В курсе покрываются следующие топики:

- Introduction to Natural Language Processing
    
- Machine Learning Basics and Text Classification
    
- Word Embeddings
    
- Convolutional Neural Networks
    
- Hidden Markov Models and Tagging
    
- Recurrent Neural Networks
    
- Topic Modeling
    
- Statistical Machine Translation
    
- Transformers
    
- Conversational AI
    

**Курсы от Стенфорда**

![](https://habrastorage.org/r/w1560/webt/ed/tt/io/edttio-5agynqcozukitmps7mcu.jpeg)

- ⭐ [**Stanford CS224N: NLP with Deep Learning**](https://web.stanford.edu/class/cs224n/) + [Video](https://youtube.com/playlist?list=PLoROMvodv4rMFqRtEuo6SGjY4XbRIVRd4&si=wGZVIPhfpEtwvJx_) + [Notes](https://vinija.ai/nlp/)
    
    Классический курс по NLP от Стенфорда, не требующий представления:
    
    1. Word Vectors
        
    2. Word Vectors and Language Models
        
    3. Backpropagation and Neural Network Basics
        
    4. Dependency Parsing
        
    5. Recurrent Neural Networks
        
    6. Sequence to Sequence Models and Machine Translation
        
    7. Transformers
        
    8. Pretraining
        
    9. Post-training (RLHF, SFT)
        
    10. Efficient Adaptation
        
    11. Question Answering
        
    12. Security and Privacy
        
    13. Benchmarking and Evaluation
        
    14. Code Generation
        
    15. Multimodal Language Models
        
    16. Human Centered NLP
        
    17. Deployment and Efficiency
        
    18. Open Questions in NLP 2024
        
- [**Stanford CS224U: Natural Language Understanding**](https://web.stanford.edu/class/cs224u/index.html)
    
- [**Stanford CS 224V: Conversational Virtual Assistants with Deep Learning**](https://web.stanford.edu/class/cs224v/schedule.html)
    

⭐ [**Hugging Face NLP course**](https://huggingface.co/learn/nlp-course/chapter0/1)

![](https://habrastorage.org/r/w1560/webt/ge/u5/yl/geu5yleerpcf-djsrbgnhj6jzym.jpeg)

Этот курс научит вас обработке естественного языка с использованием библиотек из экосистемы Hugging Face — ? [Transformers](https://github.com/huggingface/transformers), ? [Datasets](https://github.com/huggingface/datasets), ? [Tokenizers](https://github.com/huggingface/tokenizers) и ? [Accelerate](https://github.com/huggingface/accelerate), а также [Hugging Face Hub](https://huggingface.co/models):

- Главы с 1 по 4 знакомят с основными понятиями библиотеки ? Transformers. К концу этой части курса вы познакомитесь с тем, как работают модели, основанные на архитектуре Transformer, и будете знать, как запускать их из Hugging Face Hub, дообучать на своем наборе данных и делиться своими результатами в Hub!
    
- Главы с 5 по 8 обучают основам ? Datasets и ? Tokenizers, прежде чем погрузиться в классические задачи NLP. К концу этой части вы сможете самостоятельно решать наиболее распространенные проблемы NLP.
    
- Главы с 9 по 12 выходят за рамки NLP и исследуют, как модели с архитектурой Transformer можно использовать для решения задач в области обработки речи и компьютерного зрения. Попутно вы узнаете, как создавать и публиковать демо-версии своих моделей, а также оптимизировать их для production. К концу этой части вы будете готовы применить ? Transformers для решения (почти) любой задачи машинного обучения!
    

[**Linguistics for Language Technology**](https://bylinina.github.io/ling_course/)

Данный курс будет полезен тем, кто хочет не только решать NLP задачи с помощью машинного обучения, но и понимать основы лингвистической теории: слова, морфология, синтаксис, межъязыковые вариации, семантика (значения слова, значения предложения), дискурс и прагматика.

#### Книги

⭐ [**Speech and Language Processing**](https://web.stanford.edu/~jurafsky/slp3/) by Dan Jurafsky and James H. Martin

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/53d/781/2eb/53d7812eb4fc43ff88b2228852342d42.png)

Учебник, который покрывает и классические, и современные подходы от Daniel Jurafsky - это бессмертная классика, к которой постоянно выходят обновления.  
В качестве дополнения можно также посмотреть на курс [LSA 311: Computational Lexical Semantics](https://web.stanford.edu/~jurafsky/li15/) от этого же автора.

⭐ [**Natural Language Processing with Transformers**](https://transformersbook.com/) by Lewis Tunstall, Leandro von Werra adn Thomas Wolf

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/da5/192/30d/da519230d8cc050394a3ccfa1e0ac403.jpeg)

С момента своего появления в 2017 году Transformers быстро стала доминирующей архитектурой для достижения самых лучших результатов в различных NLP задачах. Эта практическая книга поможем вам:

- Создавать, отлаживать и оптимизировать модели Transformers для основных задач NLP, таких как классификация текста (Text Classification), распознавание именованных сущностей (Named Entity Recognition / NER / Token Classification) и ответы на вопросы (Question Answering).
    
- Узнать, как Transformers можно использовать для межъязыкового [трансферного обучения](https://education.yandex.ru/journal/transfernoe-obuchenie-pochemu-deep-learning-stal-dostupnee) (cross-lingual transfer learning).
    
- Применить Transformers в реальных задачах, где недостаточно размеченных данных.
    
- Делать модели Transformers эффективными для развертывания, используя такие методы, как distillation, pruning и quantization.
    
- Обучать Transformers с нуля и узнать, как это делать на нескольких GPU.
    

[**Transformers for Natural Language Processing**](https://www.amazon.com/Transformers-Natural-Language-Processing-architectures-ebook/dp/B09T34LVRM?ref_=ast_author_dp&dib=eyJ2IjoiMSJ9.wa38PcGXRxWshcH7rjbD4IcG9iwmHqu-ncXj6wO9_XMOVAx0cny9f6lqwV0McsOKUhvnNw-lAB6HtM35nxTl9z34_sglVNO4zrYs-3pvp9Y.-9Y0jF1BIRd4hjHermJekTA7kM2UDOJpGjgvguDKdpk&dib_tag=AUTHOR) by Denis Rothman

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/8d7/283/899/8d7283899a5769af541cc758d001e25e.png)

Данная книга научит вас обучать и настраивать архитектуры глубоких нейронных сетей (для NLP) с помощью Python, Hugging Face и OpenAI (GPT-3, ChatGPT и GPT-4).

#### Статьи

За список статей благодарю [Илью Гусева](https://t.me/senior_augur):

- [Word2Vec](https://arxiv.org/pdf/1301.3781.pdf), Mikolov et al., Efficient Estimation of Word Representations in Vector Space
    
- [FastText](https://arxiv.org/pdf/1607.04606.pdf), Bojanowski et al., Enriching Word Vectors with Subword Information
    
- [Attention](https://arxiv.org/abs/1409.0473), Bahdanau et al., Neural Machine Translation by Jointly Learning to Align and Translate
    
- [Transformers](https://arxiv.org/abs/1706.03762), Vaswani et al., Attention Is All You Need
    
- [BERT](https://arxiv.org/abs/1810.0480), Devlin et al., BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding
    
- [GPT-2](https://d4mucfpksywv.cloudfront.net/better-language-models/language_models_are_unsupervised_multitask_learners.pdf), Radford et al., Language Models are Unsupervised Multitask Learners
    
- [GPT-3](https://arxiv.org/abs/2005.14165), Brown et al, Language Models are Few-Shot Learners
    
- [LaBSE](https://arxiv.org/abs/2007.01852), Feng et al., Language-agnostic BERT Sentence Embedding
    
- [CLIP](https://arxiv.org/abs/2103.00020), Radford et al., Learning Transferable Visual Models From Natural Language Supervision
    
- [RoPE](https://arxiv.org/abs/2104.09864), Su et al., RoFormer: Enhanced Transformer with Rotary Position Embedding
    
- [LoRA](https://arxiv.org/abs/2106.09685), Hu et al., LoRA: Low-Rank Adaptation of Large Language Models
    
- [InstructGPT](https://arxiv.org/abs/2203.02155), Ouyang et al., Training language models to follow instructions with human feedback
    
- [Scaling laws](https://arxiv.org/abs/2203.15556), Hoffmann et al., Training Compute-Optimal Large Language Models
    
- [FlashAttention](https://arxiv.org/abs/2205.14135), Dao et al., FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness
    
- [NLLB](https://arxiv.org/abs/2207.04672), NLLB team, No Language Left Behind: Scaling Human-Centered Machine Translation
    
- [Q8](https://arxiv.org/abs/2208.07339), Dettmers et al., LLM.int8(): 8-bit Matrix Multiplication for Transformers at Scale
    
- [Self-instruct](https://arxiv.org/abs/2212.10560), Wang et al., Self-Instruct: Aligning Language Models with Self-Generated Instructions
    
- [Alpaca](https://crfm.stanford.edu/2023/03/13/alpaca.html), Taori et al., Alpaca: A Strong, Replicable Instruction-Following Model
    
- [LLaMA](https://arxiv.org/abs/2302.13971), Touvron, et al., LLaMA: Open and Efficient Foundation Language Models
    

#### Прочее

⭐ [**100 вопросов про NLP**](https://dynamic-epoch-4bb.notion.site/100-questions-about-NLP-549ccde0d81a4689b5635888b9d0d7e6)

Список из 100 вопросов (по классическому и современному NLP, а также по большим языковым моделям) поможет вам структурировать свой процесс изучения NLP и подготовки к собеседованиям.

[**Чат по NLP**](https://t.me/natural_language_processing)

Русскоязычный чат по NLP в телеграм с 6000+ участниками, в котором:

- Задают на вопросы (и иногда отвечают)
    
- Публикуют новости
    
- Анонсируют соревнования, семинары и конференции
    
- Размещают вакансии
    

### Компьютерное зрение

В данном разделе рассмотрим материалы по компьютерному зрению.

Компьютерное зрение объединяет под собой подходы к обработке и анализу изображений, где главная цель - научить компьютер воспринимать изображения как человек. Технологии на основе компьютерного зрения сейчас активно используются в промышленности, медицине, автомобильной и многих других отраслях.

[**Нейронные сети и компьютерное зрение**](https://stepik.org/course/50352/syllabus)

В этом курсе вы сделаете первые шаги в области компьютерного зрения с методами машинного обучения:

- Нейрон и нейронная сеть
    
- Строим первую нейронную сеть
    
- Задачи, решаемые при помощи нейронных сетей
    
- Методы оптимизации
    
- Свёрточные нейронные сети
    
- Регуляризация и нормализация
    
- Метод максимального правдоподобия и большой финал
    

⭐ [**CS231n: Deep Learning for Computer Vision**](http://cs231n.stanford.edu/schedule.html) + [Videos](https://www.youtube.com/watch?v=vT1JzLTH4G4&list=PL3FW7Lu3i5JvHM8ljYj-zLfQRF3EO8sYv&ab_channel=StanfordUniversitySchoolofEngineering)

![](https://habrastorage.org/r/w1560/webt/zt/u6/pd/ztu6pdj1oz890i5houlrdqegr3c.png)

Классический курс по CV от Стенфорда, не требующий представления:

- Deep Learning Basics
    
- Image Classification with Linear Classifiers
    
- Regularization and Optimization
    
- Neural Networks and Backpropagation
    
- Image Classification with CNNs
    
- CNN Architectures
    
- Training Neural Networks
    
- Recurrent Neural Networks
    
- Attention and Transformers
    
- Video Understanding
    
- Object Detection and Image Segmentation
    
- Visualizing and Understanding
    
- Self-supervised Learning
    
- Robot Learning
    
- Generative Models
    
- 3D Vision
    

[**EECS 442: Computer Vision**](https://web.eecs.umich.edu/~justincj/teaching/eecs442/WI2021/) + [Videos](https://m.bilibili.com/video/BV1BV411n7Km)

Вводный курс о компьютерном зрении.

Привожу названия тем на английском, чтобы не искажать терминологию переводом:

- Image formation / projective geometry / lighting
    
- Practical linear algebra
    
- Image processing / descriptors
    
- Image warping
    
- Linear models + optimization
    
- Neural networks
    
- Applications of neural networks
    
- Motion and flow
    
- Single-view geometry
    
- Multi-view geometry
    
- Applications
    

### Графовые нейронные сети

В данном разделе рассмотрим материалы по графовым нейронным сетям.

Графовые нейронные сети - это тип архитектуры нейронных сетей, который позволяет обучаться на графовых данных. В самом общем смысле граф — это множество точек (вершин, узлов), которые соединяются множеством линий (рёбер, дуг). Графовые нейронные сети применяются в рекомендательных системах, комбинаторной оптимизации, компьютерном зрении, физике и химии, разработке лекарств и в других отраслях.

[**CS224W: Machine Learning with Graphs**](https://web.stanford.edu/class/cs224w/) + [Video](https://www.youtube.com/playlist?list=PLoROMvodv4rOP-ImU-O1rYRg2RFxomvFp)

![](https://habrastorage.org/r/w1560/webt/yy/r1/be/yyr1bebbyomudrkjcxabag5ihkw.jpeg)

Этот курс посвящен вычислительным и алгоритмическим задачам, а также задачам моделирования, характерным для анализа больших графов. Посредством изучения базовой структуры графа и ее особенностей учащиеся знакомятся с методами машинного обучения и инструментами интеллектуального анализа данных, которые позволяют получить представление о различных сетях.

[**Graph Neural Networks**](https://aman.ai/primers/ai/gnn/)

Вводный туториал по графовым нейронным сетям.

[**Graph Neural Networks for RecSys**](https://aman.ai/recsys/gnn/)

Туториал по использованию графовых нейронных сетей в рекомендательных системах.

### Обучение с подкреплением

В данном разделе рассмотрим материалы по обучению с подкреплением.

Обучение с подкреплением — это метод машинного обучения, при котором происходит обучение модели, которая не имеет сведений о системе, но имеет возможность производить какие-либо действия в ней. Действия переводят систему в новое состояние и модель получает от системы некоторое вознаграждение.

Это не такая популярная область, как NLP или CV, но сейчас она "набирает обороты" и используется в автономных автомобилях, финансах, медицине, а также для автоматизации промышленности и в других областях.

[**Spinning Up in Deep RL**](https://spinningup.openai.com/en/latest/index.html)

![](https://habrastorage.org/r/w1560/webt/3c/o0/8q/3co08qig-ppginoiliiv-rgywg8.png)

Это образовательный ресурс, созданный OpenAI, который упрощает изучение глубокого обучения с подкреплением (deep RL).

Этот модуль содержит множество полезных ресурсов, в том числе:

- Краткое [введение](https://spinningup.openai.com/en/latest/spinningup/rl_intro.html) в терминологию RL, виды алгоритмов и базовую теорию
    
- [Эссе](https://spinningup.openai.com/en/latest/spinningup/spinningup.html) о том, как стать исследователем в RL
    
- Тщательно подобранный [список](https://spinningup.openai.com/en/latest/spinningup/keypapers.html) важных статей, организованный по темам
    
- Хорошо документированный [репозиторий](https://github.com/openai/spinningup) кода с короткими standalone реализациями ключевых алгоритмов
    
- Несколько [упражнений](https://spinningup.openai.com/en/latest/spinningup/exercises.html) для разминки
    

[**? Deep Reinforcement Learning Course**](https://huggingface.co/learn/deep-rl-course/unit0/introduction)

![](https://habrastorage.org/r/w1560/webt/uj/u0/ts/uju0tsm0kja2mlpei2yzqlxsbke.jpeg)

Этот курс научит вас глубокому обучению с подкреплением от новичка до эксперта.

В этом курсе вы:

- ? Изучите Deep Reinforcement Learning в теории и на практике.
    
- ?‍? Научитесь использовать известные библиотеки Deep RL, такие как Stable Baselines3, RL Baselines3 Zoo, Sample Factory и CleanRL.
    
- ? Обучите агентов в уникальных средах, таких как SnowballFight, Huggy the Doggo ?, VizDoom (Doom), а также в классических средах, таких как Space Invaders, PyBullet и других.
    
- ? Поделитесь своими обученными агентами с помощью одной строки кода в Hub, а также загрузите мощные агенты от сообщества.
    
- ? Примете участвуйте в соревнованиях, в которых вы будете сравнивать своих агентов с другими командами. Вы также сможете играть против агентов, которых будете обучать.
    
- ? Получите сертификат об окончании, выполнив 80% заданий.
    

[**Reinforcement Learning Textbook**](https://arxiv.org/abs/2201.09746)

![](https://habrastorage.org/r/w1560/webt/ga/90/yl/ga90ylz280pfijfu_75t3r4fjac.png)

Современные алгоритмы глубокого обучения с подкреплением способны решать задачи искусственного интеллекта методом проб и ошибок без использования каких-либо априорных знаний о решаемой задаче.

В этом конспекте собраны принципы работы основных алгоритмов, достигших прорывных результатов во многих задачах от игрового искусственного интеллекта до робототехники. Вся необходимая теория приводится с доказательствами, использующими единый ход рассуждений, унифицированные обозначения и определения. Основная задача этой работы — не только собрать информацию из разных источников в одном месте, но понять разницу между алгоритмами различного вида и объяснить, почему они выглядят именно так, а не иначе.

Предполагается знакомство читателя с основами машинного обучения и глубокого обучения.

### Рекомендательные системы

Рекомендательные системы являются довольно популярным направлением в ML и используются повсеместно, начиная от стриминговых сервисов и заканчивая маркетплейсами, то есть применяются там, где нужно порекомендовать покупателю/клиенту что-то на основании истории взаимодействия с системой всех ее пользователей.

#### Курсы

⭐ [**Your First RecSys**](https://ods.ai/tracks/mts-recsys-df2020)

![Ваша первая рекомендательная система с нуля](https://habrastorage.org/r/w1560/webt/ou/ze/dr/ouzedr8f5vab38vuvexopmot_yg.jpeg)

Ваша первая рекомендательная система с нуля

За 5 занятий вы разберетесь, как правильно поставить задачу, какие данные нужно собирать, освоите полезные приемы, попробуете популярные фреймворки для построения рекомендательных систем, создадите собственный прототип и узнаете, как довести его до продакшена:

- Введение в рекомендательные системы
    
- Методы валидации, метрики и бейзлайны
    
- Фильтрация на основе контента и коллаборативная фильтрация
    
- Градиентный бустинг на деревьях и задача ре-ранжирования
    
- Более продвинутые методы анализа данных и моделей в рекомендациях
    

⭐ [**Your Second RecSys**](https://ods.ai/tracks/recsys-course2021) (продолжение Your First RecSys)

![Продвинутая рекомендательная система](https://habrastorage.org/r/w1560/webt/ic/et/1z/icet1zni5drdzwsibelp-puml68.png)

Продвинутая рекомендательная система

На этом курсе более подробно затрагиваются проблемы оценки эффекта влияния рекомендательной модели на продукт и способы измерения этого влияния, строится production система (от файла в ноутбуке до real-time микросервисной архитектуры), и используются более сложные модели, а именно: двухуровневые и нейросетевые.  
Вас ждут лекции от разных экспертов, примеры с кодом и соревнование на реальных данных в конце курса:

1. Your first money / experiment
    
    - Бизнес-эффект от рекомендаций
        
    - Дополнительные методы оценки качества рекомендаций
        
2. Your first prod
    
    - Рекомендации в production
        
    - Ускорение рекомендаций в production
        
3. Your second model
    
    - Двухэтапная модель
        
    - Нейросетевая матричная факторизация и DSSM
        

[**Рекомендательные системы**](https://github.com/shashist/recsys-course)

Цель курса — обеспечить всестороннее введение в область рекомендательных систем:

- Первая часть курса посвящена общим подходам RecSys
    
- Во второй части кратко рассказывается о многоруких бандитах и контрфактической оценке.
    

#### Книги

⭐ К. Фальк. [**Рекомендательные системы на практике**](https://dmkpress.com/catalog/computer/data/978-5-97060-774-9/) **/** [**Practical Recommender Systems**](https://www.manning.com/books/practical-recommender-systems) by Kim Falk

![Книга "Рекомендательные системы на практике"](https://habrastorage.org/r/w1560/webt/mq/gd/ih/mqgdiharttyfjrrvaazcreaob4i.jpeg)

Книга "Рекомендательные системы на практике"

В данной книге объясняется, как работают рекомендательные системы:

- Поймете, как собирать пользовательские данные и создавать персонализированные рекомендации.
    
- Узнаете, как использовать самые популярные алгоритмы рекомендаций.
    
- Увидите их примеры в действии на таких сайтах, как Amazon и Netflix.
    
- Рассмотрите проблемы масштабирования и другие проблемы, с которыми вы можете столкнуться на практике.
    

[**Personalized Machine Learning**](https://cseweb.ucsd.edu/~jmcauley/pml/) by Julian McAuley

![Personalized Machine Learning](https://habrastorage.org/r/w1560/webt/mh/ja/ck/mhjackt3c0p17cn_0oaymemi16m.png)

Personalized Machine Learning

Довольно большой раздел этой [книги](https://cseweb.ucsd.edu/~jmcauley/pml/pml_book.pdf) (страницы 79-214) посвящен рекомендательным системам, поэтому я решил ее порекомендовать.

#### Разное

**Рекомендательные системы** в Учебнике по машинному обучению от ШАД

- [Введение в рекомендательные системы](https://education.yandex.ru/handbook/ml/article/intro-recsys)
    
- [Рекомендации на основе матричных разложений](https://education.yandex.ru/handbook/ml/article/rekomendacii-na-osnove-matrichnyh-razlozhenij)
    
- [Контентные рекомендации](https://education.yandex.ru/handbook/ml/article/kontentnye-rekomendacii)
    
- [Хорошие свойства рекомендательных систем](https://education.yandex.ru/handbook/ml/article/horoshie-svojstva-rekomendatelnyh-sistem)
    

[**Recommenders**](https://github.com/recommenders-team/recommenders)

![Recommenders](https://habrastorage.org/r/w1560/webt/4j/6a/pv/4j6apvl-27ilhovyfi3zggxx3wk.png)

Recommenders

Этот репозиторий содержит примеры и лучшие практики по созданию систем рекомендаций, представленные в виде Jupyter notebooks. В примерах подробно описана информация по пяти ключевым задачам:

- [Подготовка данных](https://github.com/recommenders-team/recommenders/blob/main/examples/01_prepare_data): Подготовка и загрузка данных для каждого алгоритма рекомендаций.
    
- [Создание модели](https://github.com/recommenders-team/recommenders/blob/main/examples/00_quick_start): Построение моделей с использованием различных рекомендательных алгоритмов, таких как Alternating Least Squares (ALS) или eXtreme Deep Factorization Machines (xDeepFM).
    
- [Оценка качества](https://github.com/recommenders-team/recommenders/blob/main/examples/03_evaluate): Оценка алгоритмов с помощью оффлайновых метрик.
    
- [Выбор модели и оптимизация](https://github.com/recommenders-team/recommenders/blob/main/examples/04_model_select_and_optimize): Поиск гиперпараметров для рекомендательных моделей.
    
- [Операционализация](https://github.com/recommenders-team/recommenders/blob/main/examples/05_operationalize): Вывод моделей в production с помощью Azure.
    

⭐ [**Рекомендательные системы**](https://education.yandex.ru/knowledge/rekomendatelnye-sistemy.-mashinnoe-obuchenie)

![Машинное обучение. Рекомендательные системы](https://habrastorage.org/r/w1560/webt/bv/2p/nb/bv2pnbbyqh5l9x5swdrb1gz33p4.jpeg)

Машинное обучение. Рекомендательные системы

Лекция К.В. Воронцова про коллаборативную фильтрацию и матричные разложения.

[**Рекомендательные системы: идеи, подходы, задачи**](https://habr.com/ru/companies/jetinfosystems/articles/453792/)

![Рекомендация похожих фильмов - одно из самых популярных применений рекомендательных систем](https://habrastorage.org/r/w1560/webt/cn/xp/of/cnxpofrpvsljwifawmg-q4shuk8.png)

Рекомендация похожих фильмов - одно из самых популярных применений рекомендательных систем

Подробная статья на Хабре, которая поможет разобраться в рекомендательных системах.

### Временные ряды

В данном разделе рассмотрим материалы по временным рядам. Материалов не очень много, поэтому, как и в нескольких предыдущих разделах, здесь не будет деления по типам.

Временные ряды используются в статистике, обработке сигналов, распознавании образов, эконометрике, финансах, прогнозировании погоды, предсказании землетрясений, электроэнцефалографии, астрономии, а также в любой области, в которой показатели меняются с течением времени.

[**Временные ряды**](https://education.yandex.ru/handbook/ml/article/vremennye-ryady)

![Пример временного ряда](https://habrastorage.org/r/w1560/webt/sb/3n/qa/sb3nqa5ozqtwvlr5pdux7rkoh6g.png)

Пример временного ряда

Статья по временным рядам из учебника по машинному обучению от ШАД, в которой сначала вводится понятие временных рядов и приводятся примеры, а затем рассказывается о сведении задачи предсказания временного ряда к задаче регрессии. Заканчивается статья разделом о декомпозиции временных рядов.

⭐ [**Topic 9. Time Series Analysis with Python**](https://mlcourse.ai/book/topic09/topic09_intro.html)

![](https://habrastorage.org/r/w1560/webt/9k/ia/v-/9kiav-tt2quqf0udagvfgj_ntw0.jpeg)

В Open Machine Learning Course, про который мы говорили во [второй статье](https://habr.com/ru/companies/megafon/articles/800919/#%D0%BA%D1%83%D1%80%D1%81%D1%8B) есть 9 глава, посвященная анализу временных рядов.

В [ней рассказывается](https://habr.com/ru/companies/ods/articles/327242/) как с ними работать в Python, какие возможные методы и модели можно использовать для прогнозирования; что такое двойное и тройное экспоненциальное взвешивание; что делать, если стационарность — это не про вас; как построить SARIMA и не умереть; и как прогнозировать xgboost-ом.

[**Прогнозирование временных рядов**](https://education.yandex.ru/knowledge/prognozirovanie-vremennyh-ryadov.-mashinnoe-obuchenie)

![](https://habrastorage.org/r/w1560/webt/0i/vu/mx/0ivumxq1kznxurduo4566xp2iia.jpeg)

Лекция К.В. Воронцова про временные ряды.

[**Time Series**](https://www.kaggle.com/learn/time-series)

Вводный курс от Kaggle Learn по временным рядам.

[**Forecasting time series with gradient boosting: Skforecast, XGBoost, LightGBM, Scikit-learn and CatBoost**](https://cienciadedatos.net/documentos/py39-forecasting-time-series-with-skforecast-xgboost-lightgbm-catboost) by Joaquín Amat Rodrigo, Javier Escobar Ortiz

![](https://habrastorage.org/r/w1560/webt/7w/p8/qi/7wp8qiraq1ahopscxdqwjamiqxk.png)

В этом гайде показано как использовать методы библиотеки skforecast для прогнозирования временных рядов с использованием моделей из библиотек XGBoost, LightGBM, Scikit-learn и CatBoost.

Груздев А.В., Рафферти Г. [**Прогнозирование временных рядов с помощью Prophet, sktime, ETNA и Greykite**](https://dmkpress.com/catalog/computer/data/978-5-93700-212-9/) `paid`

![](https://habrastorage.org/r/w1560/webt/6w/ql/jb/6wqljbwalyoolwowg8o7cwfhyuq.jpeg)

Прогнозирование – одна из задач науки о данных, которая является центральной для многих видов деятельности внутри организации. Книга посвящена популярным библиотекам прогнозирования временных рядов Prophet, Sktime, ETNA и Greykite. Разбирается математический аппарат и API каждой библиотеки. Показаны примеры решения задач прогнозирования, классификации и кластеризации временных рядов, проиллюстрированы темы конструирования и отбора признаков для временных рядов.  
В качестве примеров прогнозирования используются данные из самых разных областей – уровень углекислого газа в атмосфере, циклы солнечных пятен, количество местных осадков, число лайков в популярных соцсетях и др. Издание будет интересно специалистам по data science, регулярно решающим задачи с временными рядами.

У себя в [telegram](https://t.me/Gewissta) автор дополнительно предлагает "Прикладной анализ временных рядов в Python в 4 томах", в котором в дополнении к данной книге есть еще набор материалов "Классика бессмертна" (предварительный анализ ряда, простые модели, конструирование признаков, стратегии валидации, AR, MA, (S)ARIMA(X), ETS, VAR, TBATS/BATS, градиентный бустинг, кластеризация рядов, иерархические ряды). 1200-страничное пособие".

### Big Data

Данное направление немного выбивается из подборки, потому что характеризует не задачу, которую мы решаем, а инструмент, с помощью которого мы это делаем, но так как компании, имеющие в своем распоряжении большое количество данных, часто требуют знания этих инструментов (а именно, Spark), и спрашивают про них на собеседованиях, я решил включить этот раздел в статью.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/15a/7d0/207/15a7d02078a712b72e676ef16da6b003.jpg)

[**Анализируем данные с помощью фреймворка Spark**](https://www.youtube.com/watch?v=McXK_ObP00c) от VK

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/892/941/480/89294148019bbc639e30e629936d87d4.jpg)

Отличное поверхностное руководство по пользованию API Spark.

**Знакомство с Apache Spark** от DataLearn

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/691/119/f58/691119f58d2f2b57283ca00d67f8eead.jpg)

В 7-ом модуле курса [Введение в Инжиниринг Данных и Аналитику](https://datalearn.ru/kurs-po-getting-start-with-data-engineering) происходит знакомство с open source решением для аналитики и инжиниринга данных - Apache Spark и его коммерческой версией Databricks и аналогами Amazon Glue и Azure Synapse. Вы узнаете о примерах использования Spark в индустрии и популярные use cases. Автор расскажет о своем опыте с Apache Spark в Amazon и Microsoft и научит вас работать с данными с помощью PySpark и Spark SQL, а также поделится лучшими книгами и материалы по этой теме.

1. [Введение](https://www.youtube.com/watch?v=3qhcMVsVc5A)
    
2. [Что такое Apache Spark](https://www.youtube.com/watch?v=Tl9YzC-dQLI&t=2267s)
    
3. [Начало работы в Apache Spark](https://www.youtube.com/watch?v=FiaZnMMOV-A)
    
4. [Знакомство с Spark API](https://www.youtube.com/watch?v=gg1uip_i2KY)
    
5. [Spark SQL и операции в Spark](https://www.youtube.com/watch?v=MHt_s0-q_PM)
    

⭐ Перрен Ж.Ж. [**Spark в действии**](https://dmkpress.com/catalog/computer/data/978-5-97060-879-1/) **/** [**Spark in Action**](https://www.manning.com/books/spark-in-action-second-edition) by Jean-Georges Perrin

![](https://habrastorage.org/r/w1560/webt/7j/gy/nn/7jgynncpjvfigl-joir0abt6onc.jpeg)

Анализ корпоративных данных начинается с чтения, фильтрации и объединения файлов и потоков из многих источников. Механизм обработки данных Spark способен обрабатывать эти разнообразные объемы информации как признанный лидер в этой области, обеспечивая в 100 раз большую скорость, чем например Hadoop. Благодаря поддержке SQL, интуитивно понятному интерфейсу и простому и ясному многоязыковому API вы можете использовать Spark без глубокого изучения новой сложной экосистемы.

Эта книга научит вас создавать полноценные аналитические приложения. В качестве примера используется полный конвейер обработки данных, поступающих со спутников NASA.

[**Learning Spark**](https://pages.databricks.com/rs/094-YMS-629/images/LearningSpark2.0.pdf) by Jules S. Damji, Brooke Wenig, Tathagata Das & Denny Lee

![](https://habrastorage.org/r/w1560/webt/lm/ty/q-/lmtyq-a7nto9vbyowoaopczj36s.jpeg)

В этой книге предлагается структурированный подход к изучению Apache Spark, охватывающий новые разработки в проекте.

⭐ [**Data Analysis with Python and PySpark**](https://www.manning.com/books/data-analysis-with-python-and-pyspark) by Jonathan Rioux

![](https://habrastorage.org/r/w1560/webt/xk/re/li/xkrelia7tvc5g1hbowxyayu4y_u.jpeg)

Это ваш путеводитель по реализации успешных проектов обработки данных на основе Python. Эта практическая книга, наполненная соответствующими примерами и методами, научит вас создавать пайплайны для формирования отчетов, машинного обучения и других задач. Упражнения в каждой главе помогут вам попрактиковаться в изученном и быстро начать использовать PySpark.

## Подведем итоги

Если вы еще не читали, то рекомендую прочитать блоки [Learning How to Learn](https://habr.com/ru/companies/megafon/articles/795261/#learning-how-to-learn) и [Подведем итоги](https://habr.com/ru/companies/megafon/articles/795261/#%D0%BF%D0%BE%D0%B4%D0%B2%D0%B5%D0%B4%D0%B5%D0%BC-%D0%B8%D1%82%D0%BE%D0%B3%D0%B8) из [первой](https://habr.com/ru/companies/megafon/articles/795261/) статьи, так как все сказанное там применимо и для подготовки к секции по специализированному машинному обучению.

Собранные в этой статье материалы будут полезны при подготовке к собеседованиям на [различные позиции](https://job.megafon.ru/) в Big Data МегаФон.

А если вы только начинаете свою карьеру в Data Science, то обратите внимание на стажировки в крупных компаниях, на которых вы сможете не только прокачать свои знания, но также получить крутой опыт применения теории на практических задачах бизнеса. В МегаФоне пример такой стажировки - это акселератор (пишите на [почту](mailto:career@megafon.ru) с темой письма "_стажировка в big data_"), с помощью которого ежегодно находят свою первую работу специалисты по работе с данными (Data Scientists), аналитики (Data Analysts) и дата инженеры (Data Engineers).

## Что дальше?

В следующей статье разберем материалы для подготовки к дизайну систем машинного обучения.

Актуальные ресурсы для этой серии статей вы сможете найти в репозитории [Data Science Resources](https://github.com/Extremesarova/ds_resources), который будет поддерживаться и обновляться. В этот раз в репозиторий выложено больше материалов, чем попало в эту статью (потому что их очень много), поэтому советую зайти туда и найти то, что вам нужно.

Также вы можете подписаться на мой телеграм-канал [Data Science Weekly](https://t.me/data_science_weekly), в котором я каждую неделю делюсь интересными и полезными материалами.

Если вы знаете какие-нибудь классные ресурсы, которые я не включил в этот список, то прошу написать о них в комментариях.
