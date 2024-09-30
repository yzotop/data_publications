---
tags:
  - data
data_type:
  - AB tests
link: https://t.me/suranalytics/210
author: Balandin
company: Aviasales
source: telegram
---
Если вы задумываетесь о том, как продолжить качать свою A/B-мышцу, то этот пост для вас. 

Допустим, вы понимаете, зачем они вообще нужны, как применять стат. тесты, как их дизайнить, знаете основные ошибки (например, не будете подглядывать и будете следить за наличием SRM). 

Чтож, на самом деле, это уже почти успешный успех, скажем, 80% по принципу Парето. Но оставшиеся 20% не менее интересны и, главное, полезны. Меня вдохновляют эксперименты тем, что являются движущей силой развития продукта. Поэтому эти самые 20% могут помочь проводить эксперименты качественнее, быстрее и достовернее, помогая улучшать продукт. 

Вот, на мой взгляд, топ тем, в которые стоит погрузиться после получения хорошей базы:

## Культура A/B-тестов 

Без культуры принятия решений, основанных на экспериментах, все написанное ниже почти бесполезно.

- Про процессы (https://habr.com/ru/companies/ods/articles/716110/) на Хабр
- Культура развития A/B в Microsoft (https://www.microsoft.com/en-us/research/group/experimentation-platform-exp/articles/it-takes-a-flywheel-to-fly-kickstarting-and-keeping-the-a-b-testing-momentum/)
- В Netflix (https://netflixtechblog.com/netflix-a-culture-of-learning-394bc7d0f94c)

## Ускорение тестов

Всегда хочется проверить больше гипотез и сделать это быстрее. Мешает нам дисперсия. Основные методы ускорения тестов как раз направлены на ее сокращение. 

У меня есть два поста, где собрано то, что поможет хорошо разобраться в этой теме (при желании, даже немного закопавшись в ML): Раз (https://t.me/suranalytics/143), Два (https://t.me/suranalytics/144). 


## 🔄 Последовательное тестирование 

Существуют подходы, которые позволяют не совершать ошибку подглядывания и при этом еще и ускорять эксперименты. 

- Статья на medium (https://towardsdatascience.com/wish-tackles-peeking-with-always-valid-p-values-8a0782ac9654) про msprt
- Тут (https://rviews.rstudio.com/2019/08/22/calculating-always-valid-p-values-in-r/) есть реализация на R 

## AB без AB 

Не всегда возможно провести сплитование, тогда методы Diff-in-Diff, Casual Inference и некоторые другие вам в помощь. 

- Отличный обзор на medium (https://koch-kir.medium.com/causal-inference-from-observational-data-или-как-провести-а-в-тест-без-а-в-теста-afb84f2579f2#6685)
- То, как это делает Netflix: раз (https://netflixtechblog.com/a-survey-of-causal-inference-applications-at-netflix-b62d25175e6f), два (https://netflixtechblog.com/round-2-a-survey-of-causal-inference-applications-at-netflix-fd78328ee0bb)


## 🌐 Network effects

Иногда изменения в тесте будут влиять на контроль. Так бывает в соцсетях и в сервисах такси:

- Статья от Meta (https://medium.com/@AnalyticsAtMeta/how-meta-tests-products-with-strong-network-effects-96003a056c2c)
- Две статьи от СитиМобил: Раз (https://habr.com/ru/companies/citymobil/articles/560426/), Два (https://habr.com/ru/companies/citymobil/articles/575218/)


💡 Делитесь с коллегами и пишите в комментариях, какие еще продвинутые темы в экспериментах вы считаете важными!