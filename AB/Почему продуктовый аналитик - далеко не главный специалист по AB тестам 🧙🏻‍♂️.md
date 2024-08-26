---
tags:
  - data
data_type:
  - AB tests
link: https://t.me/suranalytics/182
source: telegram
author: Balandin
---
![[Pasted image 20240816182448.png]]

Когда ты аналитик и начинаешь изучать A/B-тесты, то в основном тебе рассказывают про статистику, анализ тестов, возможно немного про культуру экспериментов. Но это всего лишь малая часть работы над тестами, и очень много зависит от платформы экспериментов и связанного с ней. 

В некоторый момент развития компании наступает момент, когда тестов запускается столько, что требуется единая система для работы с ними. Причем техническая составляющая определенно важна, но так же важны процессы и культура, которая выстраивается вокруг платформы и помогает структурно и одинаково подходить к запуску экспериментов и контролировать достоверность их проведения. 

🧮 Из чего же состоит платформа экспериментов?

- Интеграция с кодом продукта, feature флаги. Разработчики в коде реализуют изменения в тесте и делают так, чтобы в админке в конфиге можно было настроить тест. 
- Сплитование. Деление пользователей по бакетам и экспериментам. Критичная часть для корректного проведения экспериментов. 
- Админка. Здесь запускается тест, приводятся результаты и происходит взаимодействие с тестом со сторона аналитика или продакта. 
- Хранилище данных. Тесты связаны с большим кол-вом пользовательских и технических событий, которые надо где-то хранить, да еще и оптимально.
- Витрина метрик. По сырым данным считаются метрики для каждого теста и хранятся в отдельных витринах. 
- Анализ результатов. Вся статистика, подсчет анализов тестов, их ускорение, в общем, некоторый аналитический backend. 
- Дашборды с результатами. Выводится динамика метрик и результаты тестов. 

💡А чтобы более подробно с этим разобраться, привожу статьи от топовых компаний: 
- Авито (https://habr.com/ru/companies/avito/articles/454164/)
- Ozon (https://habr.com/ru/companies/ozontech/articles/689052/)
- ОК (https://habr.com/ru/companies/odnoklassniki/articles/772958/)

- Uber: Раз, (https://www.uber.com/en-TR/blog/xp/) Два (https://www.uber.com/blog/supercharging-a-b-testing-at-uber) 
- Spotify: Раз, (https://engineering.atspotify.com/2020/10/spotifys-new-experimentation-platform-part-1/) Два (https://engineering.atspotify.com/2020/11/spotifys-new-experimentation-platform-part-2/)
- Airbnb (https://medium.com/airbnb-engineering/https-medium-com-jonathan-parks-scaling-erf-23fd17c91166) 
- Netflix (https://netflixtechblog.com/reimagining-experimentation-analysis-at-netflix-71356393af21) 
- LinkedIn: Раз, (https://www.linkedin.com/blog/engineering/ab-testing-experimentation/our-evolution-towards-t-rex-the-prehistory-of-experimentation-i) Два (https://www.linkedin.com/blog/engineering/ab-testing-experimentation/a-b-testing-variant-assignment)

Как вы думаете, надо ли аналитику разбираться в устройстве платформ экспериментов? Делитесь в комментариях!